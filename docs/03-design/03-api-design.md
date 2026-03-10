## 03 — API Design

**Traceability:** [requirements](../01-requirements/requirements.md) · [event-storming](../02-discovery/event-storming.md) · [architecture overview](./01-architecture-overview.md)  
**Tags:** [DESIGN-DEC-020] [REQ]

**OpenAPI (Swagger):** The full API is defined in OpenAPI 3.0 YAML for use in Swagger UI, Swagger Editor, or codegen: [openapi.yaml](./openapi.yaml). Import that file into [Swagger Editor](https://editor.swagger.io) or host it for Swagger UI.

---

## 1. API Principles [ADR]

- **Style:** JSON/REST over HTTPS, versioned under `/api/v1`.  
- **Multi-tenancy:** All endpoints are tenant-scoped via JWT claims and path parameters (`/tenants/{tenantId}/...`). Server validates that `tenantId` in path matches the authenticated tenant ([DESIGN-DEC-006]).  
- **Idempotency:** `PUT` and `DELETE` are idempotent; `POST` creates resources or triggers non-idempotent actions.  
- **Error model:** Standard RFC 7807-style error payloads (problem+json) for consistency.  
- **Auth:** Bearer JWT, issued by the Tenant & Access module or external IdP.

Common headers:

- `Authorization: Bearer <token>`  
- `X-Request-Id: <uuid>` (client-generated for tracing, optional)  
- `Content-Type: application/json` for JSON bodies.

---

## 2. Tenant & Access APIs [REQ]

### 2.1 Create tenant (trial signup)

- **Endpoint:** `POST /api/v1/tenants`  
- **Auth:** Public (rate-limited), email verification flow follows.  
- **Purpose:** Support “self-service deployment, trial, and purchase” ([REQ-F-013]).

Request:

```json
{
  "name": "FreshMart - MG Road",
  "admin": {
    "email": "ananya@example.com",
    "displayName": "Ananya",
    "password": "•••"
  },
  "store": {
    "name": "FreshMart MG Road",
    "type": "grocery",
    "city": "Bengaluru",
    "country": "IN"
  }
}
```

Response `201 Created`:

```json
{
  "tenantId": "tenant-abc123",
  "status": "trial",
  "plan": "trial-30d"
}
```

### 2.2 Get current tenant profile

- **Endpoint:** `GET /api/v1/tenants/{tenantId}`  
- **Auth:** Required.

---

## 3. Catalog & Inventory APIs [REQ]

Base path: `/api/v1/tenants/{tenantId}/stores/{storeId}`

### 3.1 Generate initial catalog

- **Endpoint:** `POST /api/v1/tenants/{tenantId}/stores/{storeId}/catalog:generate`  
- **Purpose:** “Product catalog was initialized” from store type and questionnaires ([REQ-F-004]).  

Request:

```json
{
  "storeType": "grocery",
  "questionnaire": {
    "hasFreshProduce": true,
    "focusBrands": ["BrandA", "BrandB"],
    "priceBand": "mid"
  },
  "maxProducts": 200
}
```

Response `202 Accepted` (async generation):

```json
{
  "jobId": "job-123",
  "status": "pending"
}
```

Client then polls:

- `GET /api/v1/jobs/{jobId}` → status + progress.  
- On completion, products appear in `GET /products`.

### 3.2 CRUD products

- **List products:**  
  - `GET /api/v1/tenants/{tenantId}/stores/{storeId}/products?search=&categoryId=&page=&pageSize=`  
- **Create product:**  
  - `POST /api/v1/tenants/{tenantId}/stores/{storeId}/products`  
  - Body aligns with `products` table (name, description, price, categoryId, etc.) [REQ-F-002].  
- **Update product:**  
  - `PUT /api/v1/tenants/{tenantId}/stores/{storeId}/products/{productId}`  
- **Delete/deactivate product:**  
  - `DELETE /api/v1/tenants/{tenantId}/stores/{storeId}/products/{productId}` (soft delete via `is_active`).

### 3.3 Gen-AI assisted product add

- **Endpoint:** `POST /api/v1/tenants/{tenantId}/stores/{storeId}/products:generate`  
- **Purpose:** “Add new products by fetching product details using Gen AI” ([REQ-F-003]).  

Request:

```json
{
  "input": "500g organic basmati rice",
  "mode": "suggest-only"
}
```

Response:

```json
{
  "suggested": {
    "name": "Organic Basmati Rice 500g",
    "description": "Long-grain aromatic rice...",
    "categoryId": "cat-rice",
    "unit": "kg",
    "price": 199.0,
    "source": "gen_ai"
  }
}
```

---

## 4. Scanner & Streaming APIs [REQ]

These endpoints are used both by the admin UI (for pairing) and by devices (for streaming).

Base path: `/api/v1/tenants/{tenantId}/stores/{storeId}`

### 4.1 Request pairing token

- **Endpoint:** `POST /api/v1/tenants/{tenantId}/stores/{storeId}/devices/pairing-tokens`  
- **Auth:** Admin JWT.  
- **Purpose:** “Scanner was paired” ([REQ-F-005], [REQ-F-007]).

Request:

```json
{
  "deviceType": "product_scanner",
  "name": "Scanner A - Entrance"
}
```

Response:

```json
{
  "deviceId": "dev-123",
  "pairingCode": "ABCD-1234",
  "expiresAt": "2025-03-02T10:30:00Z"
}
```

### 4.2 Device uses pairing token

- **Endpoint:** `POST /api/v1/devices/pair`  
- **Auth:** Device key (simple shared secret / TLS client cert for POC).  

Request:

```json
{
  "pairingCode": "ABCD-1234"
}
```

Response: `200 OK` with resolved `tenantId`, `storeId`, `deviceId`.

### 4.3 Stream inventory events (HTTP ingestion)

For POC and simple devices, HTTP ingestion is sufficient; production could add gRPC or Kafka ingestion.

- **Endpoint:** `POST /api/v1/devices/{deviceId}/inventory-events`  
- **Auth:** Device token.  

Request:

```json
{
  "events": [
    {
      "productExternalSku": "SKU-001",
      "eventType": "increment",
      "quantity": 5,
      "occurredAt": "2025-03-02T10:15:00Z"
    }
  ]
}
```

### 4.4 Stream footfall events

- **Endpoint:** `POST /api/v1/devices/{deviceId}/footfall-events`  
- **Auth:** Device token.  

Request:

```json
{
  "events": [
    {
      "gender": "female",
      "ageBucket": "26-35",
      "enteredAt": "2025-03-02T10:16:00Z"
    }
  ]
}
```

Both ingestion endpoints result in appending to `inventory_events` / `footfall_events`.

---

## 5. Display Configuration APIs [REQ]

Base path: `/api/v1/tenants/{tenantId}/stores/{storeId}`

### 5.1 List zones

- **Endpoint:** `GET /api/v1/tenants/{tenantId}/stores/{storeId}/display-zones`  
- **Purpose:** Read `display_zones` to drive visualization.  

### 5.2 Create / update zones

- `POST /display-zones`  
- `PUT /display-zones/{zoneId}`  

### 5.3 Create display configuration (weekly / ad hoc)

This refines and reuses the endpoint in `01-architecture-overview.md`.

- **Endpoint:** `POST /api/v1/tenants/{tenantId}/stores/{storeId}/display-configs`  
- **Purpose:** “Display configuration was created” ([REQ-F-009], [REQ-F-010]).

Key request fields:

```json
{
  "name": "Week 12 — Festival promo",
  "schedule": "weekly",
  "effectiveFrom": "2025-03-17T00:00:00Z",
  "useRecommendation": true,
  "sources": {
    "eventsAndTrends": true,
    "footfallLastWeek": true,
    "historicalSales": "own_db"
  },
  "zones": [
    {
      "zoneId": "entrance-left",
      "productIds": ["prod-001", "prod-002"],
      "layoutHint": "front-facing"
    }
  ]
}
```

Server behavior:

- If `useRecommendation = true`, backend calls Configuration/Recommendation Engine, which may adjust `zones` based on analytics and Gen AI.  
- Resulting layout is stored in `display_configs` + `display_layout_items` with `status = draft`.

### 5.4 Get, list, update, activate configurations

- **List:** `GET /display-configs?status=&from=&to=`  
- **Get:** `GET /display-configs/{displayConfigId}`  
- **Update:** `PUT /display-configs/{displayConfigId}` (e.g. rename, adjust layout items).  
- **Activate:** `POST /display-configs/{displayConfigId}:activate`  
  - Sets `status = active`, marks any previous active layout for that effective period as `archived`, and appends to `display_config_events`.

---

## 6. Analytics & Insights APIs [REQ]

Base path: `/api/v1/tenants/{tenantId}/stores/{storeId}`

### 6.1 Zone performance summary

- **Endpoint:** `GET /api/v1/tenants/{tenantId}/stores/{storeId}/analytics/zones`  
- **Query:** `from`, `to`, optional `displayConfigId`.  

Response (example):

```json
{
  "from": "2025-03-10",
  "to": "2025-03-17",
  "zones": [
    {
      "zoneId": "entrance-left",
      "footfallCount": 1200,
      "avgDwellSeconds": 18,
      "basketConversionRate": 0.23
    }
  ]
}
```

### 6.2 Product performance by display

- **Endpoint:** `GET /api/v1/tenants/{tenantId}/stores/{storeId}/analytics/products`  
- **Query:** `from`, `to`, optional `displayConfigId`, `productId`.  

---

## 7. API Surface Summary [DECISION]

High-level grouping of endpoints:

| Group | Example endpoints | Bounded context |
|-------|-------------------|-----------------|
| Tenant & access | `POST /tenants`, `GET /tenants/{tenantId}` | Tenant & access |
| Catalog & inventory | `POST /catalog:generate`, `GET/POST/PUT/DELETE /products` | Catalog & Inventory |
| Scanner & streaming | `POST /devices/pairing-tokens`, `POST /devices/pair`, `POST /devices/{id}/inventory-events`, `POST /devices/{id}/footfall-events` | Scanner ingestion |
| Display configuration | `GET/POST /display-zones`, `GET/POST/PUT /display-configs`, `POST /display-configs/{id}:activate` | Display configuration |
| Analytics & insights | `GET /analytics/zones`, `GET /analytics/products` | Analytics & intelligence |

> **[DESIGN-DEC-020]** This API design deliberately keeps surface area small but complete for the golden path (tenant onboarding → catalog setup → scanner pairing → weekly display optimization → analytics). Future APIs (ERP/CRM integration, advanced roles, feature flags) should extend these resource families rather than introduce unrelated patterns.


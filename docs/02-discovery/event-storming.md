# Event Storming — Retail Store Display Management App

**Traceability:** [docs/01-requirements/requirements.md](../01-requirements/requirements.md)

---

## 1. Actors List

| Actor | Description | Related requirements |
|-------|-------------|----------------------|
| Store Admin | Primary user; maintains products, inventory, scanner pairing, and display configuration via Web/Mobile. | [REQ-F-001], [REQ-F-002]–[REQ-F-010], [REQ-F-012], [REQ-NF-004] |
| Product Scanner Device | External hardware that streams inventory addition/deletion events. | [REQ-F-005], [REQ-F-006], [REQ-NF-008] |
| Main Door Scanner Device | External hardware that streams footfall data (e.g. Male/Female, approximate age). | [REQ-F-007], [REQ-F-008], [REQ-NF-008] |
| Gen AI Service | External service providing product details, initial product lists, events/trends, and historical sales context. | [REQ-F-003], [REQ-F-004], [REQ-F-010], [REQ-NF-003] |
| Configuration / Recommendation Engine | System component (LLM or custom) that proposes display configurations from events, footfall, and sales. | [REQ-F-010] |
| Analytics Engine | Incremental compute (e.g. Feldera) for data pipelines and analytics. | [REQ-NF-009] |
| ERP / CRM (future) | External systems for OEM integration; not critical for first releases. | [REQ-F-014], [REQ-NF-007] |

---

## 2. Domain Events (past-tense)

| Domain event | Description | Related requirements |
|--------------|-------------|----------------------|
| Product catalog was initialized | Initial product list generated from store type and questionnaires (synthetic/Gen AI). | [REQ-F-004] |
| Product was added | New product created; details fully or partially from Gen AI. | [REQ-F-002], [REQ-F-003] |
| Product was updated / removed | Product or inventory CRUD by store admin. | [REQ-F-002] |
| Inventory was incremented from scan | Product scanner stream recorded an inventory addition. | [REQ-F-006] |
| Inventory was decremented from scan | Product scanner stream recorded an inventory deletion. | [REQ-F-006] |
| Scanner was paired | Store admin paired product scanner or main door scanner with the store. | [REQ-F-005], [REQ-F-007] |
| Footfall was recorded | Main door scanner stream recorded a footfall (with optional demographics). | [REQ-F-008] |
| Events and trends were received | Gen AI or pipeline delivered events/trends data for display optimization. | [REQ-F-010] |
| Historical sales context was received | Gen AI or own DB provided historical sales data for configuration. | [REQ-F-010] |
| Display configuration was created | New weekly or ad hoc configuration created (LLM or custom engine). | [REQ-F-009], [REQ-F-010] |
| Display configuration was updated / activated | Store admin edited or activated a display configuration; visualization updated. | [REQ-F-009] |
| Tenant was onboarded | New store/tenant signed up (self-service trial/purchase). | [REQ-F-001], [REQ-F-013], [REQ-NF-006] |

---

## 3. Bounded Contexts Table

| Bounded context | Responsibility | Key domain events | Related requirements |
|-----------------|----------------|-------------------|----------------------|
| **Catalog & Inventory** | Product master data and inventory levels; Gen AI product enrichment; initial catalog generation. | Product catalog was initialized, Product was added/updated/removed, Inventory was incremented/decremented from scan. | [REQ-F-002], [REQ-F-003], [REQ-F-004], [REQ-F-006] |
| **Scanner ingestion** | Pairing and consuming streams from product scanners and main door scanners; normalizing and publishing events. | Scanner was paired, Inventory was incremented/decremented from scan, Footfall was recorded. | [REQ-F-005]–[REQ-F-008], [REQ-NF-008] |
| **Display configuration** | Creating, maintaining, and visualizing store display configurations; consuming events, footfall, and sales context; configuration/recommendation engine. | Events and trends were received, Historical sales context was received, Display configuration was created/updated/activated. | [REQ-F-009], [REQ-F-010], [REQ-NF-009] |
| **Tenant & access** | Multi-tenant identity, tenant onboarding, self-service trial/purchase, and access control. | Tenant was onboarded. | [REQ-F-001], [REQ-F-013], [REQ-NF-006], [ARCH-CHAR-002] |
| **Analytics & intelligence** | Pipelines and incremental computation (e.g. Feldera); aggregation of footfall, sales, and events for display engine. | Events and trends were received, Historical sales context was received (downstream). | [REQ-NF-009] |
| **Integrations (future)** | OEM-friendly APIs and adapters for ERPs and CRMs. | — | [REQ-F-014], [REQ-NF-007] |

---

## 4. Event Flow Diagrams (for PPT)

Flow diagrams below map **actors → actions → domain events** by bounded context. Use Mermaid renderer or export as PNG/SVG for slides.

### 4.1 Tenant onboarding flow

**Actors:** Store Admin · **Event:** Tenant was onboarded

```mermaid
flowchart LR
  subgraph Actor
    A[Store Admin]
  end
  subgraph Actions
    A1[Sign up / Start trial]
    A2[Verify email or phone]
    A3[Complete store profile]
  end
  subgraph Events
    E1[Tenant was onboarded]
  end
  subgraph Context
    C[Tenant & access]
  end
  A --> A1 --> A2 --> A3 --> E1
  E1 --> C
```

```mermaid
sequenceDiagram
  participant Admin as Store Admin
  participant Web as Web / Mobile App
  participant Tenant as Tenant & Access Service
  participant DB as Tenant Store

  Admin->>Web: Sign up (store name, email, phone)
  Web->>Tenant: Create tenant + trial subscription
  Tenant->>DB: Persist tenant
  Tenant->>Web: Send verification
  Web->>Admin: Verification email / SMS
  Admin->>Web: Verify link / OTP
  Web->>Tenant: Confirm verification
  Tenant->>DB: Mark tenant active
  Note over Tenant,DB: Domain event: Tenant was onboarded
  Tenant->>Web: Redirect to onboarding wizard
  Web->>Admin: Store profile form
  Admin->>Web: Submit store type, size, locale
  Web->>Tenant: Update tenant metadata
  Tenant->>DB: Save profile
```

---

### 4.2 Catalog and inventory flow

**Actors:** Store Admin, Gen AI Service · **Events:** Product catalog was initialized; Product was added/updated/removed; Inventory was incremented/decremented from scan

```mermaid
flowchart TB
  subgraph Actors
    A[Store Admin]
    G[Gen AI Service]
  end
  subgraph CatalogActions
    A1[Generate initial catalog]
    A2[Add / edit / remove product]
    A3[Receive scan stream]
  end
  subgraph Events
    E1[Product catalog was initialized]
    E2[Product was added / updated / removed]
    E3[Inventory was incremented from scan]
    E4[Inventory was decremented from scan]
  end
  subgraph Context
    C[Catalog & Inventory]
  end
  A --> A1
  G --> A1
  A1 --> E1
  A --> A2 --> E2
  A3 --> E3
  A3 --> E4
  E1 --> C
  E2 --> C
  E3 --> C
  E4 --> C
```

```mermaid
sequenceDiagram
  participant Admin as Store Admin
  participant App as Web / Mobile App
  participant Catalog as Catalog & Inventory Service
  participant GenAI as Gen AI Service
  participant Scanner as Scanner Ingestion
  participant DB as Catalog Store

  rect rgb(248,248,255)
    Note over Admin,DB: Bootstrap catalog
    Admin->>App: Start "Generate initial catalog"
    App->>Catalog: Store type + questionnaire
    Catalog->>GenAI: Request product list
    GenAI-->>Catalog: Suggested products
    Catalog->>DB: Save product list
    Note over Catalog,DB: Product catalog was initialized
    Catalog-->>App: Show list for review
    Admin->>App: Edit / add / remove items
    App->>Catalog: Update products
    Catalog->>DB: Persist
    Note over Catalog,DB: Product was added / updated / removed
  end
  rect rgb(255,248,248)
    Note over Scanner,DB: Stream from product scanner
    Scanner->>Catalog: SCAN_ADD / SCAN_REMOVE
    Catalog->>DB: Update inventory
    Note over Catalog,DB: Inventory incremented / decremented from scan
  end
```

---

### 4.3 Scanner pairing and ingestion flow

**Actors:** Store Admin, Product Scanner Device, Main Door Scanner Device · **Events:** Scanner was paired; Inventory incremented/decremented from scan; Footfall was recorded

```mermaid
flowchart TB
  subgraph Actors
    A[Store Admin]
    PS[Product Scanner Device]
    DS[Main Door Scanner Device]
  end
  subgraph Actions
    A1[Pair scanner via QR / token]
    A2[Stream scan events]
    A3[Stream footfall events]
  end
  subgraph Events
    E1[Scanner was paired]
    E2[Inventory was incremented from scan]
    E3[Inventory was decremented from scan]
    E4[Footfall was recorded]
  end
  subgraph Context
    C[Scanner ingestion]
  end
  A --> A1 --> E1
  E1 --> C
  PS --> A2 --> E2
  PS --> A2 --> E3
  DS --> A3 --> E4
  E2 --> C
  E3 --> C
  E4 --> C
```

```mermaid
sequenceDiagram
  participant Admin as Store Admin
  participant App as Web / Mobile App
  participant Ingest as Scanner Ingestion Service
  participant PS as Product Scanner Device
  participant DS as Main Door Scanner Device
  participant Catalog as Catalog & Inventory
  participant Analytics as Analytics Engine

  rect rgb(240,255,240)
    Admin->>App: Open "Connect hardware"
    App->>Ingest: Request pairing code
    Ingest-->>App: QR / registration token
    App->>Admin: Show QR for scanner
    PS->>Ingest: Register with token
    Ingest->>Ingest: Bind scanner to tenant
    Note over Ingest: Scanner was paired
    DS->>Ingest: Register with token
    Ingest->>Ingest: Bind door scanner to tenant
    Note over Ingest: Scanner was paired
  end
  rect rgb(248,255,248)
    loop Stream
      PS->>Ingest: SCAN_ADD / SCAN_REMOVE (SKU, qty)
      Ingest->>Catalog: Inventory delta
      Note over Ingest,Catalog: Inventory incremented / decremented from scan
      DS->>Ingest: FOOTFALL (timestamp, demographics)
      Ingest->>Analytics: Footfall event
      Note over Ingest,Analytics: Footfall was recorded
    end
  end
```

---

### 4.4 Display configuration flow

**Actors:** Store Admin, Gen AI Service, Configuration/Recommendation Engine, Analytics Engine · **Events:** Events and trends received; Historical sales context received; Display configuration was created/updated/activated

```mermaid
flowchart TB
  subgraph Actors
    A[Store Admin]
    G[Gen AI Service]
    R[Config / Recommendation Engine]
    AN[Analytics Engine]
  end
  subgraph DataInputs
    D1[Events and trends]
    D2[Footfall aggregates]
    D3[Historical sales]
  end
  subgraph Events
    E1[Events and trends were received]
    E2[Historical sales context was received]
    E3[Display configuration was created]
    E4[Display configuration was updated / activated]
  end
  subgraph Context
    C[Display configuration]
  end
  G --> D1
  AN --> D2
  G --> D3
  AN --> D3
  D1 --> E1
  D2 --> E1
  D3 --> E2
  E1 --> R
  E2 --> R
  R --> E3
  A --> E4
  E3 --> C
  E4 --> C
```

```mermaid
sequenceDiagram
  participant Admin as Store Admin
  participant App as Web / Mobile App
  participant Display as Display Config Service
  participant Engine as Config / Recommendation Engine
  participant Analytics as Analytics Engine
  participant GenAI as Gen AI Service
  participant DB as Display Store

  rect rgb(255,250,240)
    Analytics->>Engine: Events and trends were received
    Analytics->>Engine: Footfall aggregates
    GenAI->>Engine: Historical sales context was received
    Engine->>Engine: Compute suggested layout
    Engine->>Display: Proposed configuration
    Display->>DB: Save draft
    Note over Display,DB: Display configuration was created
  end
  rect rgb(240,255,240)
    Admin->>App: Open "Weekly suggestions"
    App->>Display: Get draft layout + KPIs
    Display-->>App: Layout + comparison
    App->>Admin: Show visualization
    Admin->>App: Edit and approve
    App->>Display: Activate layout
    Display->>DB: Mark active
    Note over Display,DB: Display configuration was updated / activated
  end
```

---

### 4.5 End-to-end event flow (high-level)

Overview of how domain events connect across bounded contexts.

```mermaid
flowchart TB
  subgraph Tenant["Tenant & access"]
    E0[Tenant was onboarded]
  end

  subgraph Catalog["Catalog & Inventory"]
    E1[Product catalog was initialized]
    E2[Product was added / updated / removed]
    E3a[Inventory was incremented from scan]
    E3b[Inventory was decremented from scan]
  end

  subgraph Scanner["Scanner ingestion"]
    E4[Scanner was paired]
    E5[Footfall was recorded]
  end

  subgraph Analytics["Analytics & intelligence"]
    E6[Events and trends were received]
    E7[Historical sales context was received]
  end

  subgraph Display["Display configuration"]
    E8[Display configuration was created]
    E9[Display configuration was updated / activated]
  end

  E0 --> E1
  E1 --> E2
  E4 --> E3a
  E4 --> E3b
  E4 --> E5
  E3a --> E6
  E3b --> E6
  E5 --> E6
  E6 --> E7
  E7 --> E8
  E8 --> E9
```

---

### 4.6 Summary table for PPT

| Flow | Diagram section | Main actors | Key events | Bounded context |
|------|-----------------|-------------|------------|-----------------|
| Tenant onboarding | 4.1 | Store Admin | Tenant was onboarded | Tenant & access |
| Catalog & inventory | 4.2 | Store Admin, Gen AI, Product Scanner | Catalog initialized, Product added/updated/removed, Inventory incremented/decremented | Catalog & Inventory |
| Scanner pairing & ingestion | 4.3 | Store Admin, Product Scanner, Door Scanner | Scanner was paired, Inventory events, Footfall was recorded | Scanner ingestion |
| Display configuration | 4.4 | Store Admin, Gen AI, Config Engine, Analytics | Events/trends received, Sales context received, Display created/updated/activated | Display configuration |
| End-to-end | 4.5 | All | All domain events | All contexts |

**Export for PPT:** Render each Mermaid block in a Markdown preview (e.g. VS Code, Cursor, or [Mermaid Live](https://mermaid.live)), then copy as PNG/SVG or use a Mermaid-to-image export tool for slides.

---

*Document generated from problem statement; traceability to [docs/01-requirements/requirements.md](../01-requirements/requirements.md).*

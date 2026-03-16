# ADR-009: API Design (REST/JSON and Tenant Scoping)

**Title:** REST/JSON over HTTPS, versioned under /api/v1; tenant-scoped paths; RFC 7807 errors  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to all tenant-facing and partner-facing HTTP APIs. OpenAPI 3.0 defines the contract; see the API design specification (`openapi.yaml`).

---

## Context

The system exposes APIs for:

- Web and Mobile clients ([REQ-F-001], [REQ-F-012]) for store admin operations.
- Future ERP/CRM and OEM integrations ([REQ-F-014], [REQ-NF-007]).

We need a consistent, secure, and maintainable API style that supports multi-tenancy (see ADR-002 Multi-tenancy strategy and ADR-007 Authentication and tenant resolution) and integrability ([ARCH-CHAR-004]).

---

## Alternatives considered

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **REST/JSON** | HTTP methods, JSON payloads, resource-oriented URLs (chosen) | Broad tooling, simple mental model, OEM-friendly; OpenAPI support | May cause over/under-fetching; multiple round-trips for complex views |
| **GraphQL** | Single endpoint, client-specified fields and nesting | Flexible queries; fewer round-trips; strong typing | Steeper learning curve for partners; caching and auth patterns differ from REST |
| **gRPC** | Binary protocol, codegen, streaming | High performance; strong contracts; streaming support | Less familiar to web/mobile and OEM partners; typically needs HTTP gateway for browser clients |
| **Version in path (/api/v1)** | Version as path segment (chosen) | Explicit breaking boundary; cache-friendly; easy to reason about | New version requires URL change and client migration |
| **Version in header/query** | e.g. `Accept-Version: 1` or `?version=1` | Same URL across versions; gradual migration | Versioning less visible; cache keys and documentation more complex |
| **Tenant in path** | `/api/v1/tenants/{tenantId}/...` (chosen) | Explicit scope; auditable; prevents path override attacks | All clients must follow URL pattern; tenant always in path |
| **Tenant in header** | e.g. `X-Tenant-Id` | Shorter URLs; tenant can be set once per client | Easier to misuse or forget; less visible in logs and proxies |
| **RFC 7807 (problem+json)** | Standard error format (chosen) | Consistent structure; tooling and client handling | Slightly larger payloads; partners must adopt the format |
| **Custom error format** | Project-specific error schema | Can be minimal or tailored | Non-standard; harder for partners to integrate; no shared ecosystem |

---

## Decision

We adopt **REST over HTTPS with JSON** and the following conventions:

- **Base path and versioning:** All public APIs live under **`/api/v1`**. Version in the path allows future v2 without breaking existing clients. No version in query or header for the primary contract.
- **Tenant scoping:** Tenant-scoped resources use the path pattern **`/api/v1/tenants/{tenantId}/...`** (and optionally `/stores/{storeId}/...`). The `tenantId` in the path MUST match the authenticated tenant from the JWT ([ADR-007]); otherwise the server returns 403. This makes tenant scope explicit and auditable.
- **HTTP methods and semantics:**  
  - **GET** — read; idempotent.  
  - **POST** — create or action (e.g. catalog:generate); may be non-idempotent.  
  - **PUT** — full replace; idempotent.  
  - **PATCH** — partial update where supported.  
  - **DELETE** — remove; idempotent.
- **Request/response:** JSON request bodies and JSON responses. `Content-Type: application/json`. Use standard HTTP status codes (200, 201, 204, 400, 401, 403, 404, 409, 500).
- **Errors:** Use **RFC 7807** problem+json style for error payloads: `type`, `title`, `status`, `detail`, optional `instance`, `requestId`, and extension fields (e.g. `tenantId`, `code`) for consistency and client handling. See the API design document for more detail.
- **Auth:** `Authorization: Bearer <JWT>`. Optional `X-Request-Id` (UUID) for correlation. No tenant id in header; tenant comes from path and is validated against JWT.
- **Idempotency:** POST for create may support `Idempotency-Key` header for critical flows (e.g. payment or tenant creation) to allow safe retries; exact usage is defined per endpoint in the OpenAPI spec.
- **Contract:** The full API is described in **OpenAPI 3.0** (e.g. `openapi.yaml`) for documentation, codegen, and validation.

This aligns with [DESIGN-DEC-005], [DESIGN-DEC-006] and supports [ARCH-CHAR-004] and [REQ-NF-007] (OEM-friendly integration points).

---

## Trade-offs

- **REST/JSON vs. richer APIs:** REST is simple and well-supported but may cause some over/under-fetching compared to GraphQL or gRPC.
- **Path versioning (/api/v1) vs. header/query:** Version-in-path makes breaking changes explicit and cache-friendly, but requires URL changes for v2+.
- **Tenant in path vs. header:** Putting tenant in the path clarifies scope and prevents misuse, at the cost of stricter URL conventions for all clients.

---

## Consequences

### Positive

- **Consistent style:** One REST/JSON approach simplifies documentation, onboarding, and tooling.
- **Tenant safety:** Path + JWT validation gives strong protection against cross-tenant access.
- **Tooling & portability:** OpenAPI and REST/JSON work well across platforms and stacks.

### Negative

- **Potential extra round-trips:** Some clients may need multiple calls or denser endpoints to avoid over/under-fetching.
- **Version churn risk:** Future breaking changes require new path versions (v2, v3), which must be managed carefully.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-F-001], [REQ-F-012], [REQ-F-014], [REQ-NF-007], [ARCH-CHAR-004]
- [ADR-002](ADR-002-multi-tenancy-strategy.md) — Multi-tenancy
- [ADR-007](ADR-007-authentication-and-tenant-resolution.md) — Auth and tenant resolution
- [docs/03-design/03-api-design.md](../03-design/03-api-design.md) — API principles and examples
- [docs/03-design/01-architecture-overview.md](../03-design/01-architecture-overview.md) — [DESIGN-DEC-005], [DESIGN-DEC-006]

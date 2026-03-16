# ADR-007: Authentication and Tenant Resolution

**Title:** JWT-based authentication; tenant from token; path tenantId must match authenticated tenant  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to all tenant-scoped API access. IdP choice (internal vs external) and trial/signup flow may be refined in implementation.

---

## Context

APIs must be **tenant-scoped** (see ADR-002 Multi-tenancy strategy) and **secure** ([ARCH-CHAR-007]). We need:

- Authentication of the caller (store admin or system).
- Resolution of the **tenant** (and optionally store and user) for every request so that data access is correctly scoped.

Requirements: [REQ-F-001] (multi-tenant SaaS), [REQ-F-013] (self-service trial/signup). Design decisions [DESIGN-DEC-005] and [DESIGN-DEC-006] in the architecture overview state: tenant from JWT or path; path `tenantId` must match authenticated tenant.

---

## Decision

We adopt **JWT-based authentication** with **tenant resolution from the token** and **strict path validation**.

- **Token:** Access to tenant-scoped APIs requires a **Bearer JWT**. The JWT is issued by our Tenant & Access module (or an external IdP we trust) and must include at least:
  - Subject (user id) and optionally email, display name.
  - **Tenant id** (e.g. `tenant_id` or `tid` claim) identifying the tenant the user belongs to.
  - Optional: `store_id` or list of store ids the user can access.
  - Expiry and issuer.
- **Tenant resolution:** For requests to `/api/v1/tenants/{tenantId}/...`, the backend:
  1. Validates the JWT and extracts the authenticated tenant id from the token.
  2. Compares `tenantId` in the path to the authenticated tenant id.
  3. If they differ, returns **403 Forbidden** (no cross-tenant access via path override).
  4. Uses the authenticated tenant id for all data access (filters, logging).
- **Public endpoints:** Only a small set of endpoints (e.g. tenant signup `POST /api/v1/tenants`, login, password reset) are unauthenticated; they are rate-limited and validated to prevent abuse.
- **Stream ingestion:** Device/scanner events are associated with tenant and store via device pairing (device belongs to a store/tenant); ingestion endpoints may use API keys or short-lived tokens scoped to a tenant/store, not end-user JWTs.

This enforces [ARCH-CHAR-002] and [ARCH-CHAR-007] and implements [DESIGN-DEC-005], [DESIGN-DEC-006].

---

## Trade-offs

- **JWT vs. simpler auth:** JWTs give us stateless auth and tenant/user identity in each request, at the cost of managing token lifecycle and size.
- **Tenant in path vs. header:** Path-based tenant scoping is explicit and auditable, but requires all clients to follow the URL pattern strictly.
- **Single-tenant session assumption:** Keeping one active tenant per session simplifies security now but may require changes if users ever span tenants.

---

## Consequences

### Positive

- **Strong tenant isolation:** Path + token checks prevent cross-tenant access by design.
- **Good observability:** Tenant and user claims in JWTs make request logging and auditing straightforward.

### Negative

- **Token management overhead:** We must define rotation/refresh strategies and handle any growth in token claims.
- **Future multi-tenant users:** Supporting users across multiple tenants would require redesign of session/selection flows.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-F-001], [REQ-F-013], [ARCH-CHAR-007]
- [ADR-002](ADR-002-multi-tenancy-strategy.md) — Multi-tenancy strategy
- [docs/03-design/01-architecture-overview.md](../03-design/01-architecture-overview.md) — [DESIGN-DEC-005], [DESIGN-DEC-006]
- [docs/03-design/03-api-design.md](../03-design/03-api-design.md) — API principles and auth headers

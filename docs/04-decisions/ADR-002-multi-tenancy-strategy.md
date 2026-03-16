# ADR-002: Multi-Tenancy Strategy

**Title:** In-process multi-tenancy with shared database and tenant-scoped data  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to all backend modules; tenant isolation must be enforced in code and verified by tests. May be augmented with DB-level RLS in a follow-up ADR if needed.

---

## Context

The system is a multi-tenant B2B SaaS ([REQ-F-001]). [ARCH-CHAR-002] Multi-tenancy and [ARCH-CHAR-007] Security & data isolation require:

- Clear tenant boundaries so one customer cannot see or modify another’s data.
- Cost-efficient shared infrastructure (no separate deployment or database per tenant at scale).
- Portability: tenant model must not lock us into a single cloud ([ARCH-CHAR-006]).

Alternatives considered:

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **Database per tenant** | Separate schema or DB per tenant | Strong isolation, simple backup/restore per tenant | Higher cost, complex migrations, harder to scale to 10K tenants |
| **Separate deployment per tenant** | One app instance per tenant | Maximum isolation | Operationally expensive; conflicts with [ARCH-CHAR-003] cost efficiency |
| **In-process, shared DB, tenant-scoped rows** | Single app and DB; every row has `tenant_id`; all access filtered by tenant | Low cost, single codebase, scales with ADR-001 | Requires strict discipline; risk of missing filter (data leak) |

---

## Decision

We adopt **in-process multi-tenancy with a shared database and tenant-scoped rows**.

- **Tenant identity:** Every tenant is identified by a UUID (`tenant_id`). Tenants own **stores**; users and data are scoped to a tenant (and often to a store).
- **Data model:** Every multi-tenant table includes `tenant_id` (and where relevant `store_id`). All queries that return or modify tenant data MUST filter by the authenticated tenant’s ID. See the data model and DB design document and [DESIGN-DEC-010].
- **Auth and API:** Tenant context is established from the authenticated user (e.g. JWT claim). API paths use `/tenants/{tenantId}/...`; the server MUST validate that `tenantId` matches the authenticated tenant. See ADR-007 (Authentication and tenant resolution).
- **Stream ingestion:** Scanner and footfall events are tagged with `tenant_id` (and `store_id`) at ingestion based on device/store pairing so that event pipelines and analytics remain tenant-scoped.
- **No per-tenant deployment:** One (or few) deployables serve all tenants; scaling is horizontal by adding instances, not by adding tenants.

This aligns with ADR-001 (Architecture style — modular monolith) and keeps [ARCH-CHAR-002], [ARCH-CHAR-003], and [ARCH-CHAR-006] viable.

---

## Trade-offs

- **Cost vs. isolation:** A single shared DB and app is cheaper to run than separate DBs/deployments, but isolation depends on correct tenant filters.
- **Discipline vs. automation:** We initially rely on code review and tests for tenant isolation, with DB-level RLS as a future hardening option.
- **Noisy neighbor vs. isolation:** Tenants share capacity; we mitigate noisy neighbors with quotas and monitoring instead of physical separation.

---

## Consequences

### Positive

- **Cost efficiency [ARCH-CHAR-003]:** Shared infra avoids proliferation of DBs and deployments.
- **Operational simplicity & portability:** One codebase and app-level tenant model simplify ops and keep us cloud-agnostic.

### Negative

- **Risk of data leakage:** A missing tenant/store filter in code can cause cross-tenant access, so tests and reviews must focus on this.
- **Coarse tenant scaling:** We scale instances, not individual tenants; noisy neighbors are handled by quotas and monitoring.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-F-001], [ARCH-CHAR-002], [ARCH-CHAR-007]
- [ADR-001](ADR-001-architecture-style.md) — Architecture style
- [ADR-007](ADR-007-authentication-and-tenant-resolution.md) — Auth and tenant resolution
- [docs/03-design/02-data-model-and-db-design.md](../03-design/02-data-model-and-db-design.md) — Tenant-scoped schema and [DESIGN-DEC-010]

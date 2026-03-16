# Architecture Decision Records (ADRs)

**Purpose:** Capture material architecture and design decisions for the Retail Store Display Management App.  
**Tags:** [ADR], [DECISION]  
**Convention:** Each ADR uses Status, Context, Decision, **Trade-offs** (bullet points), Consequences, Links.

---

## Index

| ADR | Title | Summary |
|-----|--------|---------|
| [ADR-001](ADR-001-architecture-style.md) | Architecture Style | Modular monolith with event-driven boundaries; rationale from Top 3 characteristics (Scalability, Cost efficiency, Multi-tenancy). |
| [ADR-002](ADR-002-multi-tenancy-strategy.md) | Multi-Tenancy Strategy | In-process multi-tenancy; shared database; tenant-scoped rows; tenant ID in auth and every data access. |
| [ADR-003](ADR-003-database-and-persistence.md) | Database and Persistence | Single relational store (PostgreSQL/Cloud SQL); tenant-scoped schema; append-only event tables for inventory and footfall. |
| [ADR-004](ADR-004-streaming-and-event-ingestion.md) | Streaming and Event Ingestion | HTTP/gRPC ingestion for scanner and footfall streams; canonical event schema; POC may use synthetic data. |
| [ADR-005](ADR-005-analytics-engine.md) | Analytics Engine | Incremental compute engine (e.g. Feldera) for pipelines and materialized views; subject to POC-04 validation. |
| [ADR-006](ADR-006-gen-ai-integration.md) | Gen AI Integration | External Gen AI via API for product enrichment and config context; cost guards and optional rule-based fallback. |
| [ADR-007](ADR-007-authentication-and-tenant-resolution.md) | Authentication and Tenant Resolution | JWT-based auth; tenant from token; path `tenantId` must match authenticated tenant. |
| [ADR-008](ADR-008-deployment-and-cloud-strategy.md) | Deployment and Cloud Strategy | GCP-first; stateless API and workers; managed DB; cloud-agnostic design for portability. |
| [ADR-009](ADR-009-api-design.md) | API Design | REST/JSON over HTTPS, versioned under `/api/v1`; tenant-scoped paths; RFC 7807 errors. |

---

## Traceability

- **Requirements:** [docs/01-requirements/requirements.md](../01-requirements/requirements.md)  
- **Architecture overview:** [docs/03-design/01-architecture-overview.md](../03-design/01-architecture-overview.md)  
- **Risks and POCs:** [docs/03-design/02-risks-and-pocs.md](../03-design/02-risks-and-pocs.md)

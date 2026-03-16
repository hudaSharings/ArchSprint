# ADR-008: Deployment and Cloud Strategy

**Title:** GCP-first deployment; stateless API and workers; managed DB; cloud-agnostic design  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to initial and Beta deployment. Multi-cloud or on-prem specifics (e.g. IaC for Azure) are follow-up work per the engineering roadmap (Phase 5).

---

## Context

Requirements and architecture characteristics:

- [REQ-NF-001]: Deployable on GCP initially; support for on-prem and multi-cloud (Azure, AWS) without re-architecture.
- [REQ-NF-005]: Scale 100 → 1K → 10K tenants; infra scales without redesign.
- [REQ-NF-006]: Deployment achievable via self-service; minimal manual setup.
- [ARCH-CHAR-006]: Portability (cloud-agnostic); OEM and deployment flexibility.

We need a deployment strategy that meets these while keeping cost and ops manageable ([ARCH-CHAR-003]).

---

## Decision

We adopt a **GCP-first** deployment with **stateless application components** and **managed services**, and we **avoid cloud-specific lock-in** in application code so that portability remains viable.

- **Primary platform:** First production and Beta run on **Google Cloud Platform (GCP)**. Use managed services where they reduce ops: e.g. Cloud Run or GKE for API and workers, Cloud SQL for PostgreSQL for the Store DB (see ADR-003 Database and persistence), managed monitoring and logging.
- **Stateless API and workers:** The Backend API and Stream Ingest (and optionally Config Engine and Analytics workers) are **stateless** and **horizontally scalable**. No affinity to a specific instance; tenant context comes from the request (JWT) or message (tenant_id in event). This supports [REQ-NF-005] and ADR-001 (Architecture style).
- **Database:** Managed relational DB (Cloud SQL for PostgreSQL). Use read replicas for read scaling when needed; single primary for writes. Backups and HA are managed by the platform.
- **Secrets and config:** Use a secrets manager (e.g. Secret Manager on GCP) and environment-specific config; avoid hardcoded credentials and cloud-specific APIs in business logic so that config can be swapped for another cloud or on-prem.
- **Infrastructure as Code:** Deployment and infra are defined as code (e.g. Terraform, Pulumi, or cloud-native templates) so that environments are reproducible and self-service deployment ([REQ-NF-006]) can be automated. Same patterns can be replicated for Azure or AWS in Phase 5.
- **Cloud-agnostic application design:** Prefer standard protocols (HTTP, SQL, PostgreSQL). Avoid GCP-only APIs in core business logic; use abstractions (e.g. blob storage interface, messaging interface) so that alternate implementations can be added for Azure/AWS/on-prem. Document portability decisions in ADRs ([ARCH-CHAR-006]).

---

## Trade-offs

- **GCP-first vs. multi-cloud from day one:** Starting on GCP simplifies and accelerates MVP, but shifting to other clouds later needs IaC and adapter work.
- **Managed services vs. self-hosted:** Managed compute/DB/logging reduce ops overhead while increasing dependency on GCP’s limits and pricing.
- **Stateless vs. stateful services:** Stateless workers scale easily but push all state into DB/caches, which must be carefully designed.

---

## Consequences

### Positive

- **Scalability [REQ-NF-005]:** Stateless services plus managed DB allow straightforward horizontal scaling.
- **Operational efficiency:** Managed services and IaC reduce day-to-day operational burden and support self-service deployment.

### Negative

- **Cloud dependency:** MVP depends on GCP services and quotas; future portability requires discipline in abstractions and IaC.
- **Service limits:** We must design within Cloud SQL and regional limits until additional scaling strategies are introduced.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-NF-001], [REQ-NF-005], [REQ-NF-006], [ARCH-CHAR-006]
- [ADR-001](ADR-001-architecture-style.md) — Architecture style and deployment view
- [ADR-003](ADR-003-database-and-persistence.md) — Database choice
- [docs/03-design/01-architecture-overview.md](../03-design/01-architecture-overview.md) — Deployment view (Section 7), [DESIGN-DEC-007]

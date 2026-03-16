# ADR-003: Database and Persistence Strategy

**Title:** Single relational store (PostgreSQL/Cloud SQL) with tenant-scoped schema and append-only event tables  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to Store DB and event storage for MVP and scale-up to 10K tenants. Partitioning, read replicas, or RLS may be added in follow-up ADRs.

---

## Context

The system needs a persistent store for:

- **Transactional data:** Tenants, users, stores, products, inventory snapshots, display configs, devices.
- **Event data:** Inventory events (scanner) and footfall events (door scanner) for streaming ingestion and analytics ([REQ-F-006], [REQ-F-008], [REQ-NF-008], [REQ-NF-009]).

Constraints from requirements:

- [ARCH-CHAR-003] Cost efficiency: avoid unnecessary data stores and operational cost.
- [ARCH-CHAR-002] Multi-tenancy: strict tenant isolation (see ADR-002 Multi-tenancy strategy).
- [REQ-NF-005] Scalability: 100 → 1K → 10K tenants without redesign.
- [REQ-NF-001] GCP initially; design for portability ([ARCH-CHAR-006]).

Alternatives considered:

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **Single RDBMS (Postgres)** | One database; tenant_id on all tables; event tables append-only | Simple ops, ACID, SQL, good tooling | Single store to scale; need indexing and possibly partitioning |
| **RDBMS + dedicated event store** | Separate store (e.g. Kafka, event log) for streams | Clear separation, replay | More moving parts, cost, and complexity for our scale |
| **NoSQL (e.g. document)** | Document store per tenant or shared with tenant key | Flexible schema | Less suited to relational model (tenants, stores, products, configs); analytics often need SQL |

---

## Decision

We use a **single relational database** (PostgreSQL; on GCP, Cloud SQL for PostgreSQL) as the primary persistence store.

- **Schema:** Tables are tenant-scoped per ADR-002 and the data model & DB design document. Composite indexes include `tenant_id` (and `store_id`, time) for common query patterns.
- **Transactional tables:** Standard CRUD for tenants, users, stores, products, inventory_snapshots, display_configs, display_zones, display_layout_items, devices.
- **Append-only event tables:** `inventory_events` and `footfall_events` are append-only. They feed the analytics engine (e.g. Feldera) and materialized views; updates/deletes are not used for these tables.
- **Migrations:** Schema changes are versioned and applied via migrations (e.g. Flyway, Entity Framework migrations, or similar); same process for all environments.
- **No separate event store for MVP:** Stream ingest writes directly into the RDBMS event tables (or via a thin buffer). A dedicated event bus (e.g. Kafka) can be introduced later if we need replay or multi-consumer decoupling beyond what the DB and analytics engine provide.

Managed Postgres (e.g. Cloud SQL) gives backups, HA, and scaling (read replicas) without in-house DB ops, supporting [REQ-NF-001] and [ARCH-CHAR-006] (same DB engine can run on other clouds or on-prem).

---

## Trade-offs

- **Single store vs. polyglot:** One relational DB simplifies ops and consistency, but very high volumes may require partitioning or a separate analytics store.
- **RDBMS vs. dedicated event log:** Storing events in DB tables avoids new infra, at the cost of no built-in replay/pub-sub until a log is added.
- **Managed vs. self-hosted:** Managed Postgres reduces ops work but introduces cloud service limits and some dependency on the provider.

---

## Consequences

### Positive

- **Single operational model:** One DB to back up, monitor, and tune, with strong ACID guarantees and rich SQL querying.
- **Portability & analytics:** PostgreSQL runs on major clouds/on-prem, and append-only event tables support incremental analytics per [REQ-NF-009].

### Negative

- **Scaling ceiling:** At high event volumes we may need partitioning, replicas, or a separate analytics store.
- **No native replay:** Event tables serve analytics now; a dedicated event log is needed later for full replay semantics.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-NF-001], [REQ-NF-005], [REQ-NF-008], [REQ-NF-009]
- [ADR-001](ADR-001-architecture-style.md) — Modular monolith
- [ADR-002](ADR-002-multi-tenancy-strategy.md) — Multi-tenancy strategy
- [ADR-005](ADR-005-analytics-engine.md) — Analytics engine
- [docs/03-design/02-data-model-and-db-design.md](../03-design/02-data-model-and-db-design.md) — Full schema

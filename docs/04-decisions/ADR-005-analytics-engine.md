# ADR-005: Analytics Engine (Incremental Compute)

**Title:** Use an incremental compute engine (e.g. Feldera) for analytics pipelines and materialized views  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Subject to [POC-04]: validate Feldera (or equivalent) with synthetic data for latency and ops; revisit if fit is poor ([RISK-04]).

---

## Context

The system needs analytics over:

- **Inventory events** and **footfall events** (streaming ingestion per ADR-004 Streaming and event ingestion).
- **Display configs** and **layout items** (to evaluate zone/product performance).

Outputs feed the **configuration engine** (display recommendations) and **dashboards** ([REQ-F-008], [REQ-F-010], [REQ-NF-009]).

[REQ-NF-009] states: "Analytics engine (e.g. Feldera Incremental Compute Engine) used for data pipelines and computation." We need a strategy that:

- Supports incremental computation (new events update aggregates without full recompute).
- Integrates with our Store DB and event tables ([ADR-003](ADR-003-database-and-persistence.md)).
- Stays cost-effective ([ARCH-CHAR-003]) and operable within the modular monolith story ([ADR-001](ADR-001-architecture-style.md)).

Alternatives:

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **Feldera (or similar)** | Incremental compute engine; SQL-like views over streams/tables | Incremental updates, good for real-time-ish analytics | New component; ops and fit to be validated ([POC-04]) |
| **Batch SQL / scheduled jobs** | Nightly or hourly batch over event tables | Simple, uses existing DB | Higher latency; not "streaming" analytics |
| **External analytics (e.g. BigQuery)** | Offload to cloud DWH | Scale, rich analytics | Cost, extra platform, data movement |

---

## Decision

We adopt an **incremental compute engine** (Feldera or equivalent) for analytics pipelines and materialized views.

- **Role:** Consume from event tables (or logical streams over them) and from reference data (display_configs, products); maintain materialized views such as:
  - Zone-level KPIs (e.g. `mv_zone_performance_daily`).
  - Product-level KPIs by layout (e.g. `mv_product_performance_weekly`).
- **Integration:** Engine runs as part of the backend footprint (e.g. same deployment or worker process); reads from Store DB (or a CDC/log stream). Results can be written back to the Store DB as materialized view tables or served via API.
- **Incremental semantics:** Prefer incremental computation so that new events update aggregates without full table scans; supports near–real-time display recommendations and dashboards.
- **Validation:** [POC-04] will validate Feldera (or an alternative) with synthetic footfall and inventory events; success criteria: pipelines run, latency and ops acceptable for Beta. If Feldera does not fit, we will document an alternative (e.g. lightweight stream processor or batch SQL) in a follow-up ADR.

This decision supports [REQ-NF-009] and [REQ-F-010] (configuration engine driven by footfall and sales/analytics).

---

## Trade-offs

- **Incremental engine vs. batch SQL:** We gain fresher analytics and lower recompute cost than nightly jobs, at the price of a new component to operate.
- **Integrated vs. external analytics:** Keeping analytics within our backend footprint simplifies ops but may require offloading to a DWH at very large scale.
- **Engine choice risk:** Targeting Feldera (or similar) speeds us up now but depends on [POC-04] to validate fit; we may need an alternative later.

---

## Consequences

### Positive

- **Meets analytics needs:** Satisfies [REQ-NF-009] and supports timely data for recommendations and dashboards.
- **Efficient updates:** Incremental compute avoids full recomputes, helping both performance and cost.

### Negative

- **New dependency:** We rely on the chosen engine’s stability and performance; [RISK-04] and [POC-04] must validate this.
- **Operational overhead:** Adds another piece of infrastructure to deploy, monitor, and upgrade.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-NF-009], [REQ-F-008], [REQ-F-010]
- [ADR-001](ADR-001-architecture-style.md) — Event-driven boundaries
- [ADR-003](ADR-003-database-and-persistence.md) — Event tables and Store DB
- [ADR-004](ADR-004-streaming-and-event-ingestion.md) — Event ingestion
- [docs/03-design/02-risks-and-pocs.md](../03-design/02-risks-and-pocs.md) — [RISK-04], [POC-04]

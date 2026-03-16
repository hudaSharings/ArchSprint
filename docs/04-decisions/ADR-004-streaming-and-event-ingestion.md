# ADR-004: Streaming and Event Ingestion

**Title:** HTTP/gRPC ingestion for scanner and footfall streams; canonical event schema; POC may use synthetic data  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Applies to product scanner and main door (footfall) scanner ingestion. Protocol details may be refined when real device specs are available ([RISK-01]).

---

## Context

The system must consume:

- **Product scanner streams:** Inventory additions and deletions ([REQ-F-005], [REQ-F-006]).
- **Main door scanner streams:** Footfall with attributes (e.g. gender, approximate age) ([REQ-F-007], [REQ-F-008]).

Data is streaming ([REQ-NF-008]); POC may use synthetic data. These streams feed inventory state, analytics, and the display configuration engine (see ADR-001 Architecture style), [REQ-NF-009].

Requirements and risks:

- [REQ-NF-008]: Streaming ingestion from product and door scanners.
- [ARCH-CHAR-004]: Integrability — clear APIs and adapters for external systems.
- [RISK-01] in [docs/03-design/02-risks-and-pocs.md](../03-design/02-risks-and-pocs.md): Scanner protocol/format may differ by vendor; we need a canonical internal model.

---

## Decision

We adopt the following for scanner and footfall ingestion:

- **Ingestion interface:** Provide **HTTP (REST)** endpoints for pushing events. Devices (or an edge gateway) send batches or single events to the backend. We do not require devices to connect to a message broker directly; the backend owns the ingestion API and can buffer/write to the Store DB or internal event bus. gRPC endpoints MAY be added later if we identify clear performance or integration benefits.
- **Canonical event schema:** Define internal event types (e.g. `InventoryEvent`, `FootfallEvent`) with required fields: `tenant_id`, `store_id`, `device_id` (where applicable), timestamps, and payload (e.g. product_id, quantity, event_type for inventory; gender, age_bucket for footfall). Real scanner payloads are mapped to this schema via adapters.
- **Tenant and store context:** Every event is tagged with `tenant_id` (and `store_id`) from device/store pairing so that downstream processing and analytics remain tenant-scoped (see ADR-002 Multi-tenancy strategy).
- **Persistence:** Ingested events are written to append-only tables (`inventory_events`, `footfall_events`) in the Store DB (see ADR-003 Database and persistence). The analytics engine (e.g. Feldera) consumes from these or from a logical stream built on top of them.
- **POC / synthetic data:** For POC, we support **synthetic event generators** that produce events conforming to the canonical schema so that backend and analytics can be built and tested without real hardware ([POC-01]).
- **Event-driven boundaries:** Once events are stored, internal modules (Catalog & Inventory, Analytics, Configuration Engine) react via reads, materialized views, or internal events—keeping the ingestion path decoupled from business logic per ADR-001.

---

## Trade-offs

- **HTTP vs. streaming protocols:** HTTP ingestion is simpler for devices and backend control, but has higher latency than broker-based streaming.
- **Canonical schema vs. vendor diversity:** A single internal event model stabilizes pipelines while requiring adapters for each scanner type.
- **Synthetic vs. real data early:** Synthetic generators de-risk design without hardware; we still need validation with real devices before production.

---

## Consequences

### Positive

- **Vendor flexibility:** New scanners can integrate via adapters into the same canonical event model.
- **Testability:** Synthetic streams support end-to-end testing of ingest and analytics, reducing [RISK-01].
- **Architectural fit:** Matches our event-driven boundaries and shared DB, supporting [REQ-NF-008] and [REQ-NF-009].

### Negative

- **Higher latency:** HTTP-based ingest is slower than dedicated streaming protocols, though acceptable for this domain today.
- **Adapter overhead:** Each new vendor requires adapter work to fit the canonical schema.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-F-005]–[REQ-F-008], [REQ-NF-008]
- [ADR-001](ADR-001-architecture-style.md) — Event-driven boundaries
- [ADR-002](ADR-002-multi-tenancy-strategy.md) — Tenant scoping of events
- [ADR-003](ADR-003-database-and-persistence.md) — Append-only event tables
- [docs/03-design/02-risks-and-pocs.md](../03-design/02-risks-and-pocs.md) — [RISK-01], [POC-01]

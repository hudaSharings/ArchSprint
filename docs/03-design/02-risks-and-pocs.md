# 02 — Risks and POCs

**Purpose:** Capture knowns, unknowns, and prioritized POCs for alignment and de-risking.  
**Tags:** [RISK], [POC]  
**Traceability:** [requirements](../01-requirements/requirements.md) | [ADR-001](../04-decisions/ADR-001-architecture-style.md)

---

## 1. Knowns

What we have agreed or established from the architecture sprint.

| Area | Known | Source |
|------|--------|--------|
| **Scope** | Multi-tenant B2B SaaS; Web + Mobile (Android/iOS); store admin as primary user | [REQ-F-001], [REQ-F-012] |
| **Architecture style** | Modular monolith with event-driven boundaries for streams and analytics | ADR-001 |
| **Top 3 characteristics** | Scalability, Cost efficiency, Multi-tenancy (prioritized) | [ARCH-CHAR-001], [ARCH-CHAR-002], [ARCH-CHAR-003]; requirements §3.2 |
| **Platform** | GCP-first; design allows on-prem and multi-cloud (Azure, AWS) | [REQ-NF-001], [ARCH-CHAR-006] |
| **Bounded contexts** | Catalog & Inventory, Scanner ingestion, Display configuration, Tenant & access, Analytics & intelligence | [event-storming](../02-discovery/event-storming.md) |
| **Data entry points** | Product scanner streams (inventory add/remove); main door scanner streams (footfall); Gen AI for product details, events, sales context | [REQ-F-005]–[REQ-F-008], [REQ-F-003], [REQ-F-010] |
| **Analytics** | Incremental compute engine (e.g. Feldera) for pipelines; feeds display config engine | [REQ-NF-009] |
| **Multi-tenancy** | In-process; tenant ID in auth and all data access; shared DB, tenant-scoped rows | ADR-001, [ARCH-CHAR-002] |
| **Growth targets** | 100 tenants (6 months post-Beta), 1K (next 6 months), 10K (following year) | [REQ-NF-005] |
| **GTM** | Self-service deployment, trial, and purchase; India first; multi-language later | [REQ-F-013], [REQ-NF-006], [REQ-F-011] |

---

## 2. Unknowns

Open questions, assumptions to validate, and uncertainties. Resolve via POCs, spikes, or stakeholder alignment.

| ID | Unknown | Impact if wrong | Mitigation |
|----|---------|------------------|------------|
| [RISK-01] | **Scanner protocol / format** — Real product and door scanner APIs/formats not yet fixed; may differ by vendor | Integration rework; adapter layer needed | [POC-01] synthetic stream; define canonical event schema early |
| [RISK-02] | **Gen AI provider and cost** — Which provider(s), rate limits, and cost per tenant for product enrichment + events/sales | [REQ-NF-003] at risk; trials may be too expensive | [POC-02] benchmark one provider; set usage caps and caching strategy |
| [RISK-03] | **LLM vs custom config engine** — Whether display recommendations use LLM or rule-based engine; quality vs cost | [REQ-F-010] scope and NFR cost | [POC-03] compare LLM vs rules for one scenario (e.g. weekly layout) |
| [RISK-04] | **Feldera fit** — Feldera (or alternative) for incremental pipelines at our scale and ops model | [REQ-NF-009]; swap cost if unfit | [POC-04] run Feldera (or alternative) with synthetic footfall + sales; measure latency and ops |
| [RISK-05] | **Tenant isolation verification** — No formal test strategy yet for “no cross-tenant data leak” | [ARCH-CHAR-002], [ARCH-CHAR-007] | Design tenant-id-in-query contract; add POC test suite for isolation |
| [RISK-06] | **Self-service sign-up flow** — Billing, trial limits, and activation flow not designed | [REQ-F-013], [REQ-NF-006] | Align with product; optional [POC-05] for sign-up + trial gate |
| [RISK-07] | **Mobile parity** — Whether first release is Web-only or Web + Mobile; affects timeline | [REQ-F-012] | Product decision; document as assumption once decided |

---

## 3. Prioritized POCs

Proof-of-concepts to de-risk unknowns. Ordered by priority (1 = highest).

| Priority | ID | POC | Objective | Success criteria | Resolves |
|----------|----|-----|-----------|------------------|----------|
| 1 | [POC-01] | **Synthetic scanner & footfall streams** | Produce canonical event streams (inventory add/remove, footfall) for backend and analytics without real hardware | Stream ingest consumes events; analytics pipeline runs; schema documented | [RISK-01], [REQ-F-006], [REQ-F-008] |
| 2 | [POC-02] | **Gen AI cost & quality** | One Gen AI provider for product enrichment and/or events/sales; measure cost per tenant and quality | Cost per tenant within trial budget; acceptable quality for product add and config context | [RISK-02], [REQ-NF-003] |
| 3 | [POC-03] | **Configuration engine (LLM vs rules)** | Compare LLM-driven vs rule-based display recommendation for one scenario | Recommendation quality and latency acceptable; cost fits [REQ-NF-003] | [RISK-03], [REQ-F-010] |
| 4 | [POC-04] | **Analytics engine (Feldera or alternative)** | Run incremental pipelines (e.g. footfall aggregates, sales context) with synthetic data | Pipelines run; latency and ops acceptable for Beta | [RISK-04], [REQ-NF-009] |
| 5 | [POC-05] | **Tenant isolation** | Automated tests: every query scoped by tenant; no cross-tenant read/write | Test suite passes; documented tenant-id contract | [RISK-05], [ARCH-CHAR-002], [ARCH-CHAR-007] |

---

## 4. Summary

- **Knowns:** Scope, architecture style, Top 3 NFRs, platform, bounded contexts, data sources, multi-tenancy model, and growth/GTM targets are aligned.
- **Unknowns:** Scanner interfaces, Gen AI cost/provider, config engine approach, analytics engine fit, tenant isolation verification, self-service flow, and mobile scope need resolution or explicit assumptions.
- **POCs:** Prioritized order is (1) synthetic streams, (2) Gen AI cost/quality, (3) config engine (LLM vs rules), (4) analytics engine, (5) tenant isolation tests. Run in this order to unblock backend and product build.

---

*Links: [requirements](../01-requirements/requirements.md) | [ADR-001](../04-decisions/ADR-001-architecture-style.md) | [event-storming](../02-discovery/event-storming.md) | [architecture overview](01-architecture-overview.md)*

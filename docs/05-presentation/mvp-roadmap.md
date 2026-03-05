# MVP Roadmap — Retail Store Display Management App

**Purpose:** Timeline and phases from architecture complete to Beta and scale.  
**Traceability:** [docs/01-requirements/requirements.md](../01-requirements/requirements.md) (goals: 100 → 1K → 10K customers).

---

## Timeline Table

| Phase | Timeframe | Objectives | Key deliverables | Success criteria |
|-------|-----------|------------|------------------|------------------|
| **1. Design & Architecture** | Weeks 1–2 | Lock problem, scope, and architecture | Design sprint outcomes; C1/C2; event storming; ADR-001; requirements baselined | Stakeholder sign-off on requirements and architecture style |
| **2. Data & Backend MVP** | Weeks 3–8 | Tenant model, APIs, streaming ingest, analytics plumbing | Multi-tenant API (catalog, inventory, display config); stream ingest (product + footfall, synthetic if needed); Store DB schema; analytics pipeline (e.g. Feldera) | API supports CRUD and scanner/footfall events; tenant isolation verified |
| **3. Frontend & Config Engine** | Weeks 9–14 | Web + Mobile UIs; configuration/recommendation engine | Web app (products, inventory, display config, visualization); Mobile app (Android/iOS); config engine (LLM or custom) consuming events, footfall, sales | Store admin can onboard, manage products, and create/visualize display configs; Gen AI integration working |
| **4. Beta** | Weeks 15–20 | Self-service trial, first paying customers | Self-service sign-up, trial, and purchase; monitoring and cost controls; first 100 customers | 100 tenants onboarded; trial-to-paid flow live; NFRs (usability, cost) met |
| **5. Scale (1K → 10K)** | Months 6–24 | Growth to 1K then 10K tenants; international readiness | Scaling (API, workers, DB); multi-language (India first); ERP/CRM integration hooks; Series A readiness | 1K tenants (month 12); 10K tenants (month 24); platform stable and portable |

---

## Phase Summary

| Phase | Focus |
|-------|--------|
| 1 | Design Sprint + Architecture Sprint → requirements, event storming, C1/C2, ADR-001 |
| 2 | Data engineering (synthetic/real streams), backend API, tenant model, analytics engine |
| 3 | Web & Mobile front ends, display configuration UI, configuration engine, Gen AI integration |
| 4 | Beta release, self-service GTM, 100 customers, cost and usability validation |
| 5 | Scale to 1K then 10K, multi-language, OEM readiness, international expansion |

---

## References

- **Goals:** docs/01-requirements/requirements.md (growth targets, [REQ-NF-005], [REQ-F-013])
- **Architecture:** docs/04-decisions/ADR-001-architecture-style.md; diagrams/container-c2.mmd
- **Domain:** docs/02-discovery/event-storming.md

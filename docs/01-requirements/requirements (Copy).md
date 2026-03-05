# Retail Store Display Management App — Requirements

**Source:** Problem statement (Design/Architecture Sprint)  
**Conventions:** [REQ] = requirement; tables use [REQ-XXX] for traceability.

---

## 1. Functional Requirements

| ID | Requirement | Source / Notes |
|----|-------------|----------------|
| [REQ-F001] | The system SHALL provide a **multi-tenant SaaS** offering with Web and Mobile (Android & iOS) front ends. | Solution: multi-tenant SAAS, Web and Mobile |
| [REQ-F002] | As a store admin I SHALL be able to **maintain a list of products and inventory** (CRUD). | Use case: maintain products and inventory |
| [REQ-F003] | As a store admin I SHALL be able to **add new products** by fetching product details using Gen AI (fully or partially). | Use case: Add products via Gen AI |
| [REQ-F004] | As a store admin I SHALL be able to **generate an initial product list** based on type of store and guided questionnaires, using synthetic data and/or Gen AI. | Use case: initial product list from store type + questionnaires |
| [REQ-F005] | As a store admin I SHALL be able to **pair the software with product scanners** that stream data for inventory additions and deletions. | Use case: pair with product scanners |
| [REQ-F006] | The system SHALL **consume data from product scanners** to maintain inventory (streaming); POC may use synthetic data for streams. | Use case: System consumes product scanner data |
| [REQ-F007] | As a store admin I SHALL be able to **pair with main door scanners** that stream footfalls with attributes (e.g. Male/Female, approximate age). | Use case: pair with main door scanners |
| [REQ-F008] | The system SHALL **consume data from main door scanners** and use footfall intelligence in designing store displays. | Use case: System consumes footfall for display design |
| [REQ-F009] | As a store admin I SHALL be able to **create and maintain store display configuration** along with **visualization**. | Use case: create/maintain display config + visualization |
| [REQ-F010] | As a store admin I SHALL be able to **create a new display configuration** weekly or ad hoc, driven by: events data from Gen AI, last week footfall data, and historical sales (Gen AI or own DB); configuration MAY be driven by an LLM or custom engine. | Use case: weekly/ad hoc config with events, footfall, sales |
| [REQ-F011] | The system SHALL support **multi-language** (initial target: India; long-term: worldwide). | Technical/business: multi-language, India first |
| [REQ-F012] | The system SHALL support **Android and iOS** mobile clients. | Technical/business: Android and iOS |
| [REQ-F013] | The system SHALL support **self-service deployment**, trial, and purchase (Go To Market). | Technical/business: self-service, trial, purchase |
| [REQ-F014] | The system SHALL support **integration with ERPs and CRMs**; not critical for first few releases. | Technical/business: OEM, ERP/CRM integration |

---

## 2. Non-Functional Requirements

| ID | Category | Requirement | Source / Notes |
|----|----------|-------------|----------------|
| [REQ-NF001] | Platform | The solution SHALL be built initially on **GCP**, with flexibility for **on-prem** and **multi-cloud** (Azure, AWS). | Partnership with Google Cloud; on-prem and multi-cloud flexibility |
| [REQ-NF002] | Cost | **Pricing SHALL be cost-effective** to suit price-sensitive small stores (B2B). | B2B, small stores, price sensitive |
| [REQ-NF003] | Cost | **Gen AI and cloud costs SHALL be managed** to enable cost-effective trials and low ongoing operations cost. | Cost-effective trials, low cost of operations |
| [REQ-NF004] | Usability | The **Web and Mobile UI SHALL be intuitive and easy to use** for non-technical store admins. | Main use is store admin; non-technical persons |
| [REQ-NF005] | Scalability | The system SHALL support growth targets: **100 customers** in first 6 months post-Beta, **1000** in next 6 months, **10000** in the following year; infra SHALL scale accordingly. | Goal: 100 → 1000 → 10000 customers; Series A then international |
| [REQ-NF006] | Deployment | Deployment SHALL be **easy (preferably self-service)** to support low marketing cost and adoption. | Self-service deployment, Go To Market |
| [REQ-NF007] | Integrability | The system SHALL be **OEM-friendly** for ERP vendors; integration points for ERPs and CRMs SHALL be considered in design. | Niche solution, OEM potential |
| [REQ-NF008] | Data | The system SHALL consume **streaming data** from product scanners and main door scanners; POC may use **synthetic data** for streams and product/events data. | Product scanner stream, footfall stream, synthetic data |
| [REQ-NF009] | Analytics | An **analytics engine** (e.g. Feldera Incremental Compute Engine) SHALL be used for data pipelines and computation. | POC: Data Engineering, Feldera |

---

## 3. Architecture Characteristics: Top 7 → Top 3

### 3.1 Top 7 Candidate Characteristics

| # | Characteristic | Rationale |
|---|----------------|-----------|
| 1 | **Scalability** | Growth from 100 → 10,000 customers and future international expansion demand horizontal scaling and capacity planning. |
| 2 | **Multi-tenancy** | Core to SaaS model; isolation, tenant-specific config, and fair resource usage are essential. |
| 3 | **Cost efficiency** | Critical for price-sensitive B2B segment, low-cost trials, and sustainable unit economics. |
| 4 | **Integrability** | Many external systems: product scanners, door scanners, Gen AI, ERPs/CRMs, analytics (e.g. Feldera); clear APIs and adapters required. |
| 5 | **Usability** | Primary users are non-technical store admins; UX directly impacts adoption and support cost. |
| 6 | **Portability / Cloud-agnostic** | GCP first, with requirement for on-prem and Azure/AWS; avoid lock-in and enable OEM/deployment flexibility. |
| 7 | **Security & data isolation** | B2B multi-tenant; tenant data isolation, access control, and compliance (e.g. India, then global) are mandatory. |

### 3.2 Top 3 Architecture Characteristics (with justification)

| Rank | Characteristic | Justification |
|------|----------------|----------------|
| **1** | **Scalability** [REQ] | Growth targets (100 → 1000 → 10000) and post–Series A international scaling make scalability a first-order constraint. Architecture must support elastic scaling, multi-region readiness, and cost-aware scaling so that adding tenants does not require re-architecture. |
| **2** | **Cost efficiency** [REQ] | Business model targets price-sensitive small stores; expensive solutions increase CAC and limit adoption. Cost efficiency drives: lean Gen AI usage, efficient cloud spend, multi-tenancy for shared infra, and design choices that keep trials and ongoing ops affordable. |
| **3** | **Multi-tenancy** [REQ] | SaaS is explicitly multi-tenant. Multi-tenancy affects data model, security boundaries, configuration, and operations. Getting tenant isolation and resource fairness right early avoids costly refactors and supports OEM and multi-cloud deployment. |

### 3.3 Summary

- **Scalability** — Ensures the system can grow with customer targets and future international rollout.
- **Cost efficiency** — Ensures viability for the target segment and sustainable unit economics.
- **Multi-tenancy** — Ensures correct SaaS foundation for isolation, security, and deployment flexibility.

These three characteristics should be reflected in C1/C2, physical architecture, and ADRs (e.g. scaling strategy, cost controls, tenant model and isolation).

---

*Document version: 1.0 — Generated from problem statement for Architecture Sprint.*

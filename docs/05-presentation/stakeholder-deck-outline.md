# Stakeholder Deck Outline — Retail Store Display Management App

**Purpose:** High-level presentation for stakeholders (exec, product, engineering).  
**Slides:** 10 | **Source:** Architecture sprint artefacts in `docs/` and `diagrams/`.

---

## Slide 1: Title & Agenda

**Title:** Retail Store Display Management App — Architecture & Roadmap

**Key bullet points:**
- Problem and solution in one sentence
- What we’re covering: problem, solution, requirements, domain model, architecture, MVP roadmap
- Outcome: shared understanding and go-ahead for next phase

**Source:** —
---

## Slide 2: Problem Statement

**Title:** Why We’re Building This

**Key bullet points:**
- Store display drives sales; today it’s mostly experience-based and manual
- Gen AI can encode best practices and scale factors (events, weather, trends, footfall)
- Small stores are price-sensitive; need low-cost, self-service, trial-friendly GTM
- Goal: 100 → 1,000 → 10,000 customers; then international / Series A

**Source:** Problem statement (summarised in docs/01-requirements/requirements.md).
---

## Slide 3: Solution Overview

**Title:** What We’re Building

**Key bullet points:**
- Multi-tenant B2B SaaS: Web + Mobile (Android & iOS) for store admins
- Product & inventory with Gen AI enrichment; scanner streams for inventory and footfall
- Store display configuration with visualization; weekly/ad hoc configs driven by events, footfall, and sales (LLM or custom engine)
- Self-service sign-up, trial, and purchase; GCP-first, multi-cloud/on-prem ready

**Source:** docs/01-requirements/requirements.md (Functional Requirements table); docs/02-discovery/event-storming.md (Actors).
---

## Slide 4: Requirements at a Glance

**Title:** Requirements Summary

**Key bullet points:**
- **Functional:** Multi-tenant SaaS, product/inventory CRUD, Gen AI product add & initial list, scanner pairing & stream consumption, display config & visualization, self-service trial/purchase
- **Non-functional:** GCP-first + portable, cost-effective pricing & Gen AI spend, scalable to 10K tenants, self-service deployment, streaming ingestion, analytics engine (e.g. Feldera)
- **Top 3 architecture characteristics:** Scalability, Cost efficiency, Multi-tenancy

**Source:** docs/01-requirements/requirements.md — Sections 1 (Functional), 2 (Non-Functional), 3.2 (Top 3).
---

## Slide 5: Domain Model & Bounded Contexts

**Title:** Domain Model — What We’re Managing

**Key bullet points:**
- **Actors:** Store Admin, Product Scanners, Main Door Scanners, Gen AI Service, Configuration Engine, Analytics Engine
- **Bounded contexts:** Catalog & Inventory | Scanner ingestion | Display configuration | Tenant & access | Analytics & intelligence | Integrations (future)
- Domain events (past tense): e.g. Product catalog was initialized, Inventory was incremented from scan, Footfall was recorded, Display configuration was created

**Source:** docs/02-discovery/event-storming.md — Actors list, Domain Events, Bounded Contexts table.
---

## Slide 6: System Context (C1)

**Title:** Who Uses the System and What It Talks To

**Key bullet points:**
- **User:** Store Admin (Web & Mobile)
- **Our system:** Retail Store Display Management App (multi-tenant SaaS)
- **External systems:** Product Scanners (stream), Main Door Scanners (stream), Gen AI Service (API), ERP/CRM (future)

**Source:** See C1 diagram: diagrams/context-c1.mmd (or export PNG for slide).
---

## Slide 7: Container View (C2)

**Title:** Main Building Blocks

**Key bullet points:**
- **Front ends:** Web App, Mobile App (Android/iOS) → Backend API
- **Backend:** API, Stream Ingest, Configuration Engine, Analytics Engine, Store DB
- Scanners and door scanners feed Stream Ingest; Gen AI used by Configuration Engine; analytics supports display recommendations

**Source:** See C2 diagram: diagrams/container-c2.mmd (or export PNG for slide).
---

## Slide 8: Architecture Style & Top 3

**Title:** Why Modular Monolith + Event-Driven

**Key bullet points:**
- **Chosen style:** Modular Monolith with event-driven boundaries for streams and analytics
- **Scalability:** Stateless API + scalable stream/analytics workers; DB scales with replicas/partitioning
- **Cost efficiency:** Fewer deployables than microservices; shared DB; controlled Gen AI/analytics spend
- **Multi-tenancy:** Single codebase; tenant ID everywhere; clear isolation and fair usage

**Source:** docs/04-decisions/ADR-001-architecture-style.md; docs/01-requirements/requirements.md Section 3.2 (Top 3).
---

## Slide 9: MVP Roadmap

**Title:** MVP Roadmap & Milestones

**Key bullet points:**
- Phase 1: Design & Architecture (sprint outcomes, ADRs, C1/C2)
- Phase 2: Data & backend MVP (pipelines, API, tenant model, scanner/footfall ingest)
- Phase 3: Frontend & config engine (Web/Mobile, display config, Gen AI integration)
- Phase 4: Beta (self-service trial, first 100 customers)
- Phase 5: Scale (1K → 10K tenants, international readiness)

**Source:** docs/05-presentation/mvp-roadmap.md (timeline table).
---

## Slide 10: Next Steps & Links

**Title:** Next Steps and Key Documents

**Key bullet points:**
- Finalise API and DB design; lock POC tech stack (GCP services, Feldera, Gen AI provider)
- Kick off Data Engineering and Product build per MVP roadmap
- **Links:** requirements (docs/01-requirements/requirements.md) | event storming (docs/02-discovery/event-storming.md) | ADR-001 (docs/04-decisions/ADR-001-architecture-style.md) | C1 (diagrams/context-c1.mmd) | C2 (diagrams/container-c2.mmd) | MVP roadmap (docs/05-presentation/mvp-roadmap.md)

**Source:** All docs/ and diagrams/ referenced above.
---

*Use this outline to build the actual slide deck; embed or reference the listed artefacts.*

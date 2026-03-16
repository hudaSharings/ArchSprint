# ADR-006: Gen AI Integration

**Title:** External Gen AI via API for product enrichment and display config context; cost guards and optional fallback  
**Tags:** [ADR], [DECISION]

---

## Status

**Accepted** — Provider choice and exact usage caps to be set after [POC-02] and [POC-03]. Architecture decision (external API, bounded usage) is fixed.

---

## Context

Gen AI is used for:

- **Product enrichment** ([REQ-F-003]): Fetch or complete product details (name, description, category) when adding products.
- **Initial catalog generation** ([REQ-F-004]): Generate product lists from store type and questionnaires using synthetic data and Gen AI (either or both, based on scenario).
- **Display configuration** ([REQ-F-010]): Events, trends, footfall, and historical sales context may be enriched or proposed by an LLM or custom engine.

Constraints:

- [REQ-NF-003]: Gen AI and cloud spend bounded for cost-effective trials and low ongoing cost per tenant.
- [ARCH-CHAR-004]: Integrability — clear integration point for Gen AI.
- **Data protection:** Personally identifiable information (PII), tenant identifiers, and fine-grained transactional details MUST NOT be sent to external Gen AI; only generalized product context and aggregated signals are allowed.

Risks: [RISK-02] (provider and cost), [RISK-03] (LLM vs rule-based config engine). POCs [POC-02] and [POC-03] address these.

Alternatives considered:

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **Managed external Gen AI API** | Use a cloud Gen AI provider over HTTPS (current primary choice) | Fast to adopt; no model hosting; strong models and ecosystem | Data leaves our infra; subject to provider latency, availability, and pricing |
| **Self-hosted LLM (e.g. Ollama)** | Run open models inside our own infra | Data never leaves VPC; full control over retention and locality | Requires GPU/CPU capacity, model ops, monitoring, and upgrades; models may lag best managed offerings |
| **No Gen AI (rules only)** | Use rule-based heuristics for all features | No Gen AI cost or data risk; simple to operate | Weaker UX and automation; falls short of Gen-AI-enabled product vision |

---

## Decision

We implement a **provider-agnostic Gen AI interface** and support both **external managed APIs** and **self-hosted models**, with strict cost and data-privacy guards.

- **Integration pattern:** The backend (Configuration Engine and Catalog modules) calls Gen AI through an internal `GenAiService` abstraction. This can be backed by an external managed API (e.g. OpenAI, Vertex AI, or regional equivalent) or by a self-hosted LLM runtime (e.g. Ollama or similar) deployed inside our infra.
- **Data shaping and privacy:** Only non-identifying product context and aggregated analytics signals are sent to external Gen AI. Tenant identifiers, store addresses, user IDs, and raw transactional details are never sent outside; if such data is required for a use case, it must be processed via self-hosted models or rules instead.
- **Product enrichment ([REQ-F-003], [REQ-F-004]):** Optional Gen AI step when adding or generating products: call Gen AI with minimal input (e.g. generic product descriptors, category, store type) to get name, description, or full product stub. Results are stored with a `source` flag (e.g. `gen_ai`, `manual`, `generated_template`) for traceability. Usage per tenant or per request can be capped (rate limit, monthly quota).
- **Display config ([REQ-F-010]):** Configuration Engine may use Gen AI to propose layouts from events, footfall, and sales context—or use a **rule-based engine** as fallback or default. [POC-03] will compare LLM vs rules; cost-sensitive or privacy-sensitive deployments may prefer rules or self-hosted Gen AI, while others may use managed APIs.
- **Cost guards:** Per-tenant or global limits (e.g. API calls per day, tokens per month); caching of Gen AI responses where applicable; monitoring and alerts on spend. This supports [REQ-NF-003] and viable trials.
- **Failure and fallback:** If Gen AI is unavailable or quota exceeded, product flows fall back to manual entry or template; config engine falls back to rules or cached recommendation. No hard dependency on Gen AI for core flows.

---

## Trade-offs

- **External API vs. self-hosted models:** External Gen AI avoids hosting models ourselves but ties us to provider latency, availability, and pricing.
- **Cost control vs. feature depth:** Usage caps and caching keep [REQ-NF-003] on track, but limit how broadly we can apply Gen AI.
- **LLM vs. rule-based engine:** Rule-based recommendations are cheaper and predictable; LLM-based ones may be better but costlier and less stable.

---

## Consequences

### Positive

- **Cost control:** Bounded usage and fallbacks keep unit economics and trials viable ([REQ-NF-003], [ARCH-CHAR-003]).
- **Flexibility & traceability:** Provider-agnostic integration and tracked “source” fields enable multi-region options and quality review.

### Negative

- **External dependency:** API latency/availability issues directly affect user flows, so robust fallbacks are mandatory.
- **Output quality:** LLM responses need validation; [POC-02] and [POC-03] guide where rules are preferable.

---

## Links

- [docs/01-requirements/requirements.md](../01-requirements/requirements.md) — [REQ-F-003], [REQ-F-004], [REQ-F-010], [REQ-NF-003]
- [docs/03-design/02-risks-and-pocs.md](../03-design/02-risks-and-pocs.md) — [RISK-02], [RISK-03], [POC-02], [POC-03]

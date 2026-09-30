# AI / Machine Learning — Core Track Notes

**Purpose:** The professional track of this diary. Everything here exists to answer one question: **what does it take to be an AI Engineer in Economics/Business** — not "what model should I use", but "what is the whole system around the model, and which business decision does it serve".

**Scope boundary:** this folder holds *concepts and architecture*. Code lives in the sibling repositories (see below). No note here duplicates a code walkthrough.

**Code repository:** [NguyenPhan161206/Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) · `git@github.com:NguyenPhan161206/Snake_ANN.git`

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | [AI in Business: Overview](01-ai-in-business.md) | B (Business) | Which business problems does AI actually fit — and which is it just trend-chasing? |
| 02 | [Search Systems: Lexical, Dense, Hybrid](02-search-systems-lexical-dense-hybrid.md) | A (Tech) | How do you retrieve the right chunk when the user's words and the document's words differ? |
| 03 | [RAG & Learning-to-Rank in Recruitment](03-rag-and-learning-to-rank.md) | A (Tech) | How do you rank retrieved evidence so the best answer lands at position 1? |
| 04 | [XP Engineering Practices for ML](04-xp-engineering-practices-for-ml.md) | A (Tech) | How do you test a system whose most common failure is *silently wrong*? |
| 05 | [MLOps Lifecycle & Deployment Architecture](05-mlops-lifecycle-and-deployment-architecture.md) | A (Tech) | What breaks between "the model works on my laptop" and "it works for users"? |
| 06 | [MCP & Function Calling — Agent Interfaces](06-mcp-and-function-calling-agent-interfaces.md) | A (Tech) | How does an LLM call your system safely, and how do you audit it? |
| 07 | [Affective Computing & Emotion Recognition](07-affective-computing-and-emotion-recognition.md) | A (Tech) | How do you detect a user's emotional state — and why are the published accuracy numbers not a target? |

---

## 🗺️ Reading order

The notes are numbered chronologically, but the *conceptual* order is not the same. Read in this order:

1. **01** — the business frame. Without it, notes 02–06 are a bag of techniques with no reason attached.
2. **05** — the production frame, read *before* the technique notes. It defines the vocabulary the others borrow: feature store, model registry, serving engine, monitoring, drift. Reading it third (as the numbering suggests) means reading notes 02–04 without knowing what they are building toward.
3. **02 → 03** — retrieval, then ranking. 02 explains *how candidates are fetched*, 03 explains *how they are ordered*. They are one system split across two notes; 03 assumes 02.
4. **04** — engineering discipline. The counterweight to 02/03: an ML system that cannot be tested and shipped is a demo.
5. **06** — the newest surface. Agent interfaces re-use everything above (retrieval as a tool, ranking as a tool result, MLOps as the deployment target) and add the governance layer.

**If you only read three:** 01, 05, 06 — business case, production reality, and the interface layer where the 2026 job descriptions are pointing.

---

## 🧩 How the notes fit together

```
Business pain (01)
      ↓  data, standardised
   Data layer ──────────────► Feature store (05)
      ↓
   Retrieval (02) ──► Ranking (03) ──► Generation + grounding
      ↓                                    ↓
   Engineering discipline (04) ────► Deployable, testable, monitorable (05)
      ↓
   Agent interface: retrieval & ranking become *tools* (06)
```

Notes 04 and 05 are the two that turn the others from a tutorial into a product. If you ever feel you understand 02/03 but cannot explain what a job posting means by "MLOps", that gap is 05.

---

## 🔗 Tech Cross-References (Section K)

- **Data structures** — the retrieval structures in 02/03 *are* the DSA track applied to embeddings: hash maps for the lexical index, heaps or order-statistics trees for top-$k$, incremental structures for ANN indexes → [`../dev_foundation/dsa/05-data-structure-selection-guide.md`](../dev_foundation/dsa/05-data-structure-selection-guide.md), [`../dev_foundation/dsa/04-complexity-analysis-toolkit.md`](../dev_foundation/dsa/04-complexity-analysis-toolkit.md)
- **Databases** — a feature store is a schema-design problem with a staleness SLA; vector search lives in PostgreSQL with `pgvector` → [`../dev_foundation/databases/README.md`](../dev_foundation/databases/README.md)
- **YAML & config** — model configs, serving configs, and CI pipelines are all config files; a malformed `serving.yaml` is a production incident → [`../dev_foundation/yaml_config/README.md`](../dev_foundation/yaml_config/README.md)
- **Linux & Docker** — the serving box is a machine you deploy onto; model containers are how "the same model" survives three environments → [`../dev_foundation/linux/README.md`](../dev_foundation/linux/README.md), [`../dev_foundation/docker/README.md`](../dev_foundation/docker/README.md)
- **Software architecture** — serving architecture, async boundaries, and the modular-monolith-vs-microservices choice all reappear in 05 → [`../software_architecture/README.md`](../software_architecture/README.md)
- **Economics & business** — the bridge back to the career goal: network effects explain why a model provider wins, and opportunity cost explains why a fine-tune is sometimes not worth it → [`../economics_business/04-network-effects.md`](../economics_business/04-network-effects.md), [`../economics_business/02-opportunity-cost.md`](../economics_business/02-opportunity-cost.md)

---

## 🔗 Related Topics

- [`../dev_foundation/README.md`](../dev_foundation/README.md) — the engineering foundation this track is built on
- [`../iot_aiot/01-rate-limit-and-sse.md`](../iot_aiot/01-rate-limit-and-sse.md) — the streaming/rate-limit layer an inference endpoint must respect
- [`../software_architecture/02-modular-monolith-vs-microservices.md`](../software_architecture/02-modular-monolith-vs-microservices.md) — where a model service belongs
- [`../vocabulary/README.md`](../vocabulary/README.md) — the terminology this folder reuses heavily (embedding, inference, drift)
- 🔗 [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) — code & explanations live there; this diary holds concept anchors and roadmap

---

## 📐 Template

Notes 02–07 use **Template A: Tech Note**; note 01 uses **Template B: Business Analysis** because it is a business-framing note, not a technique note. Every note must pass the Section H depth checklist for its template.

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Prompt engineering as a *system* (not a trick) | It appears in every note here but has no home; needed before 06 can be practised |
| High | Evaluation & observability for LLM outputs (offline benchmarks vs online metrics) | 05 covers infrastructure monitoring, not answer quality; the gap between them is where LLM projects actually fail |
| Medium | Fine-tuning vs prompting vs RAG — the decision | Directly serves the "opportunity cost" argument; currently scattered across 02/03 |
| Medium | Embedding model selection & vector dimensionality trade-offs | 02 asserts dense retrieval is better; nobody has explained the cost side yet |
| Medium | Cost & latency engineering for inference (batching, quantisation, caching) | Turns 05's architecture into numbers a business can budget for |
| **High** | **Sentiment vs. emotion vs. toxicity — the taxonomy nobody agrees on** | Three overlapping literatures with three label sets; note 07 assumes you can tell them apart and then finds the boundary problem unresolved |
| **Medium** | **Multimodal learning: fusion, alignment, missing modalities** | Note 07 §2.2 covers fusion strategies; the *alignment* half (getting text and image into a shared space) is the missing piece any multimodal engineer actually implements |
| Low | Multi-tenancy & data isolation for AI SaaS | Only relevant once a deployable product exists |
| Low | A bridge note: AI in Economics specifically (market microstructure, demand forecasting) | The stated specialisation; currently only note 01 touches the domain |

---

*Last updated: 2026-09-30*

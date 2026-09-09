# RAG Systems and Learning-to-Rank (LTR) Models in Recruitment

**Date Created:** 2026-09-07
**Tags:** `#rag`, `#learning-to-rank`, `#lambdamart`, `#cross-encoder`, `#information-retrieval`, `#llm`
**Reference/Source:** Graduation Thesis (Vo Ngoc Tram Anh, IT Cohort 2022-2026)

---

## 📌 Key Takeaways (TL;DR)
- **Learning to Rank (LTR):** Uses algorithms such as LambdaMART and Cross-Encoder to optimize candidate display ranking, prioritizing top-rank precision (evaluated by NDCG).
- **RAG Architecture:** Combines Information Retrieval and LLM Generation to evaluate candidates based on actual context, minimizing unsubstantiated hallucinations.
- **Data & System Management:** Dual-layer storage design (In-memory for fast retrieval, PostgreSQL JSONB for flexible result storage) and asynchronous architecture (FastAPI) to minimize latency.
- **Feature Engineering:** Combines both lexical signals (BM25) and semantic signals (Vector Embedding, TF-IDF) along with resume structure (experience, education) to train LTR models.

## 🧠 Detailed Content

### 1. Classification and Ranking Optimization
- **Label Space:** $Y = \{0, 1, 2\}$ corresponding to Reject, Hold, and Shortlist (Proceeding to the next round).
- **Ranking Relation:** 
  - $d_i > d_j \Leftrightarrow f(q, d_i) > f(q, d_j)$
  - If scores are tied, apply tie-breaking rules.
- **Optimization Objective:** Uses **NDCG@k** as the primary evaluation metric. DCG (Discounted Cumulative Gain) heavily penalizes errors at top-$k$ positions because recruiters primarily examine candidates at the top of the list.

### 2. RAG (Retrieval-Augmented Generation) Architecture
Combines non-parametric memory (retrieval system) and parametric memory (LLM) across 3 stages:
1. **Indexing:** Chunking, normalization, and vector storage.
2. **Retrieval:** Extracting the most relevant documents based on matching.
3. **Generation:** LLM generates responses conditioned on the retrieved documents.

*To constrain hallucination, the system utilizes resume data in JSON format and matching snippets, strictly bounding the LLM's reasoning space.*

### 3. Multi-tier Learning-to-Rank (LTR) Model
Optimizes the relative order among documents.

**a. LambdaMART:**
- Based on Gradient Boosted Decision Trees.
- Uses $\lambda$-gradients to approximate the optimization direction for non-differentiable metrics like NDCG.
- Learns via pairwise comparisons and combines multiple signal sources (lexical, semantic, structural).

**b. Re-ranking with Cross-Encoder:**
- Resolves the limitations of Bi-Encoders (which encode queries and documents independently).
- Concatenates query and document into a single sequence, using self-attention in Transformers to capture deep token-level interactions.
- Computationally expensive; applied only to a small preliminary retrieved candidate pool (retrieve-then-rerank).

**c. Feature Engineering for LTR (Hybrid Retrieval):**
Integrates 4 primary feature groups:
- **Group A (Foundational Signals):** Normalized Okapi BM25 score, Semantic vector similarity.
- **Group B (Skill Matching):** Hard skill Jaccard similarity, Soft skill TF-IDF, Mandatory requirement coverage ratio.
- **Group C (Resume Structure):** Seniority level, Years of experience, Education level.
- **Group D (Auxiliary Lexical Signals):** BM25 score on job title, TF-IDF of hard skills.

### 4. Evaluation Metrics
- **Ranking (Retrieval):** 
  - **NDCG:** Position-aware metric that prioritizes relevant documents at higher ranks.
  - **MAP:** Mean Average Precision.
  - **MRR:** Mean Reciprocal Rank (position of the first relevant result).
- **Text Generation (RAGAS framework):**
  - **Faithfulness:** Factual consistency, avoiding fabrication outside the provided context.
  - **Answer Relevancy:** How pertinent the response is to the query.
  - **Context Precision:** Accuracy and usefulness of the retrieved context.

### 5. Serving & System Architecture Design (Model Serving)
- **Dual-Layer Storage Management:**
  1. *In-memory / File-based:* Stores static indexes (BM25 Index) in RAM to minimize I/O latency.
  2. *Relational Database (PostgreSQL):* Uses `JSONB` fields to store semi-structured (denormalized) LLM outputs, allowing easy prompt iteration without database schema migrations.
- **Asynchronous Architecture:** Uses FastAPI (Async/Await) to invoke third-party LLM APIs (GPT-4, Claude) without blocking system worker threads.

---

## 🤔 Open Questions / Further Research Topics
*Directions to explore more deeply in the near future:*
- [ ] How does the $\lambda$-gradients calculation mechanism specifically work in the LambdaMART algorithm?
- [ ] How to implement (code) a Cross-Encoder alongside a Bi-Encoder model in a re-ranking pipeline.
- [ ] How to set up the RAGAS library to automate evaluations of Faithfulness and Answer Relevancy for LLM responses.
- [ ] How to optimize In-memory Indexing in Python to achieve high-speed computation for Okapi BM25 ranking functions?

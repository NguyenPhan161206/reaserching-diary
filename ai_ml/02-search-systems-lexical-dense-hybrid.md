# Search Systems: Lexical, Dense, and Hybrid Search

**Date Created:** 2026-09-04
**Knowledge Update Date:** 2026-09-07
**Tags:** `#AI`, `#NLP`, `#SearchEngine`, `#VectorDatabase`, `#InformationRetrieval`
**References/Sources:** [Gemini Share](https://share.gemini.google/zEAVGArmeM7e) & [https://share.gemini.google/C3HUVEIvYfGX]

---

## 📌 Key Takeaways (TL;DR)
- **Lexical Retrieval (BM25):** Traditional keyword-based search method. Extremely fast and explainable, but poor at understanding context and synonyms.
- **Dense Retrieval (Vector Search):** Uses AI embedding models to understand semantics (semantic-based). Completely solves misspellings and synonyms.
- **Hybrid Search:** Combines the strengths of both Lexical and Dense retrieval via algorithms like RRF, ensuring the system does not miss exact technical keywords while still understanding user intent.

## 🧠 Detailed Content

### 1. Lexical Retrieval & The BM25 Algorithm
**Lexical Retrieval (or Sparse Retrieval)** is the mechanism behind traditional search engines, operating on exact keyword matching.
- **Strengths:** Extremely low latency, minimal compute resource usage, clear explainability. Essential for looking up error codes, IDs, and specialized proper nouns.
- **Weaknesses:** Suffers from the "vocabulary mismatch" problem—unable to match if the query uses "automobile" while the document uses "car". Struggles with typos and misspellings.

**BM25 Algorithm (Best Matching 25):** 
The go-to ranking algorithm (the core of Elasticsearch/Lucene) for Lexical Retrieval. Scores documents based on three core factors:
1. **Term Frequency (TF):** The frequency of keyword occurrences. Uses a saturation function to prevent keyword stuffing from inflating scores by repeating a word hundreds of times.
2. **Inverse Document Frequency (IDF):** The rarity of a term. Overly common words (stop words) are down-weighted, while rare or specialized words are boosted with higher scores.
3. **Document Length:** Penalizes overly long documents to maintain fairness regarding keyword density compared to shorter documents.

### 2. Embeddings and Dense Retrieval
**What are Embeddings?**
Computers do not understand raw text. Embedding is the process of using AI models (such as OpenAI Text-Embedding, BERT) to compress/convert a piece of text into a **Vector** (a multi-dimensional array of real numbers).
- The breakthrough is that concepts with similar meanings lie close to each other in vector space (mathematizing semantics).
- It is called **Dense** because the vectors are relatively compact (typically 384–1536 dimensions), yet every dimension holds a meaningful real-valued number. This contrasts with Lexical methods that generate massive **Sparse** vectors (equal to the vocabulary size) dominated by zeros.

**Dense Retrieval Workflow (Vector/Semantic Search):**
1. **Indexing:** Pass all raw documents through an embedding model -> Store them in a **Vector Database**.
2. **Querying & Similarity Search:** The user inputs a query -> Pass it through the same embedding model to obtain a vector -> Calculate the distance/similarity (e.g., using *Cosine Similarity*) to retrieve the top $k$ nearest results, even if there is no exact keyword overlap.

### 3. Hybrid Search (The Best of Both Worlds)
Modern search systems (such as Google) rely on Hybrid Search.
- **Why is Hybrid needed?** Consider the query: `"How to fix OOM-KILLER error"`. 
  - If using only Vector Search: It might return articles on "how to resolve out-of-memory errors" (semantically relevant, but missing the critical technical keyword OOM-KILLER). 
  - If using only BM25: It might return a spammy log file repeatedly mentioning the word OOM-KILLER rather than a helpful "how-to guide".

**Approach:** 
Execute both Lexical and Dense pipelines in parallel. The Lexical branch acts as a "gatekeeper" anchoring exact keywords, while the Dense branch captures overarching semantic intent.

**Fusion and Re-ranking (RRF):**
The system cannot directly sum the two types of scores (since the scales of BM25 and Cosine similarity are completely different). Instead, it applies the **RRF (Reciprocal Rank Fusion)** algorithm:
- Ignores absolute scores.
- Focuses solely on the **Rank** of each document within each branch. 
- A document ranking near the top in both lists receives a substantial score boost, rising to the most optimal final display position.

## 🚀 Deep Dive: Additional Insights (Updated 2026-09-07)

### 1. Mathematical Foundation of BM25 and Parameter Tuning (k1, b)
The original BM25 scoring formula for document $D$ and query $Q$:
$$score(D, Q) = \sum_{q_i \in Q} IDF(q_i) \cdot \frac{TF(q_i, D) \cdot (k_1 + 1)}{TF(q_i, D) + k_1 \cdot (1 - b + b \cdot \frac{|D|}{avgdl})}$$
In Elasticsearch/OpenSearch, you can tune two main parameters:
- **$k_1$ (Term frequency saturation):** Typically defaults to 1.2. It controls how quickly term frequency saturation is reached as a keyword appears multiple times.
  - Increasing $k_1$ (e.g., 2.0): Scores continue to increase significantly as the term repeats (suitable for long-form text like articles/papers).
  - Decreasing $k_1$ (e.g., 0.1): Emphasizes whether the term is *present* or not; repeating it 100 times will not significantly boost the score.
- **$b$ (Length normalization):** Typically defaults to 0.75. Controls the penalty applied to longer documents.
  - Increasing $b$ towards 1: Heavily penalizes long documents. Suitable when documents are expected to be very short (e.g., titles, tweets).
  - Decreasing $b$ towards 0: Disables document length normalization. Suitable when documents vary widely in length, but length does not reflect noise or dilution.

### 2. ANN (Approximate Nearest Neighbor) Algorithms in Vector Databases
To search across billions of vectors without performing exhaustive comparisons (KNN), Vector DBs use ANN algorithms, trading a small amount of accuracy (Recall) for ultra-fast query execution (low Latency).
- **HNSW (Hierarchical Navigable Small World):** A multi-layer graph structure (similar to a Skip List). The upper layers have sparse connections, allowing the algorithm to take long leaps across the vector space. As it approaches the target, it descends to denser lower layers to find precise matches. HNSW provides excellent speed and Recall, serving as the current industry standard.
- **FAISS (Facebook AI Similarity Search):** Renowned for its **IVF-PQ** technique. It partitions the space into clusters (Voronoi cells - Inverted File Index) to search only within the closest clusters. Simultaneously, it uses Product Quantization (PQ) to compress vectors from float32 into compact byte codes, drastically reducing RAM consumption.

### 3. Architectures of Popular Vector Databases
- **Pinecone:** Fully-managed, cloud-native SaaS solution. Extremely easy to use with zero infrastructure maintenance, but it is closed-source and incurs usage-based costs.
- **Milvus:** A massive distributed ecosystem. Fully decouples storage and compute/query layers. Ideal for enterprises handling massive scale (billions of vectors) requiring complex clusters.
- **Qdrant:** Written in Rust. Stands out with its "Payload-based Filtering" capability, seamlessly combining traditional metadata filtering (SQL-like) with vector search without degrading search performance.
- **ChromaDB:** Written in Python/TypeScript. Lightweight and frequently used as an embedded library (In-memory/Local SQLite) during development and prototyping (PoC) for LLM/LangChain applications, with client-server support for modest scaling.

### 4. Cross-Encoder Setup for the Re-ranking Step in Hybrid Search
Bi-Encoders (used to generate vectors for Dense Search) are fast because vectors are precomputed, but the model remains "blind" to cross-interactions between individual query terms and document terms.
To achieve a breakthrough in search accuracy, the most widely adopted architecture is **Retrieve-then-Rerank**:
- **Stage 1 (Retrieve):** Use Hybrid Search (Bi-Encoder + BM25) at ultra-high speed to fetch the top 100 candidate documents.
- **Stage 2 (Re-rank):** Pass the query and these 100 documents through a **Cross-Encoder** model. Unlike Bi-Encoders, a Cross-Encoder takes input formatted as `[CLS] Query [SEP] Document [SEP]` and processes it across Transformer layers.
- **Mechanism:** Thanks to the Self-Attention mechanism, every token in the query directly interacts cross-attentively with every token in the document. Although computationally expensive (much slower), applying it to only 100 candidates keeps latency within real-time thresholds while delivering dramatic improvements in overall accuracy (MAP, NDCG).

## 📚 Recommended References
- Primary Source: [https://share.gemini.google/zEAVGArmeM7e](https://share.gemini.google/zEAVGArmeM7e)
- Supplementary Source: [https://share.gemini.google/C3HUVEIvYfGX](https://share.gemini.google/C3HUVEIvYfGX)

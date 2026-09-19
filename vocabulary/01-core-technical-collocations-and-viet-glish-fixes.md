# Vocabulary & Collocation — Core Technical Terms & Viet-Glish Fixes

**Created Date:** 2026-09-19
**Last Updated:** 2026-09-19
**Tags:** `#vocabulary`, `#it`, `#ai_ml`, `#git_github`
**Status:** Published

---

## 📌 Term List & Quick Reference

| Term / Phrase | IPA | Core Meaning | Domain |
|---------------|-----|--------------|--------|
| **Score / Rank** | `/skɔːr/ /ræŋk/` | To evaluate and order search results or items | AI / Search Systems |
| **Optimize** | `/ˈɑːptɪmaɪz/` | To make a system/algorithm as effective as possible | Software Engineering |
| **Loophole** | `/ˈluːpˌhoʊl/` | A weakness or gap in a system security/logic | System Design / Security |
| **Route** | `/ruːt/` | To direct or send data packets/messages along a path | Networking / IoT / APIs |
| **Fetch** | `/fetʃ/` | To request and retrieve data from a remote server/database | Web Development / Databases |
| **Query** | `/ˈkwɪri/` | A structured request for information from a database/search engine | Databases / Information Retrieval |
| **Deploy** | /dɪˈplɔɪ/ | To release code or models into production environments | DevOps / MLOps |
| **Keyword-stuffing** | /ˈkiːwɜːrd ˈstʌfɪŋ/ | Spamming excessive keywords to artificially inflate search rank | Search Engines / SEO / AI Bias |
| **Visualize** | /ˈvɪʒuəlaɪz/ | To form a mental image or graphical representation | Software Architecture |
| **Convert** | /kənˈvɜːrt/ | To transform data from one format/representation to another | Data Engineering / AI Embeddings |
| **Compensate** | /ˈkɑːmpənseɪt/ | To offset or make up for individual system weaknesses | System Design / Hybrid Architecture |
| **Strategy** | /ˈstrætədʒi/ | A high-level plan or orientation to achieve long-term goals | Business / AI Engineering |
| **Release** | /rɪˈliːs/ | To officially deploy code, models, or products to production/users | DevOps / Software Engineering |
| **Align / Alignment** | `/əˈlaɪn/ /əˈlaɪnmənt/` | To adjust or match systems/models to human intents or specifications | AI / System Architecture |
| **Spike** | `/spaɪk/` | A time-boxed technical research task in Agile or sudden metric surge | Agile / MLOps / Systems |

---

## 🧠 Detailed Vocabulary Analysis

### 1. Score & Rank
- **IPA:** `/skɔːr/`, `/ræŋk/`
- **Definition & Domain Context:** In search systems and recommendation engines, scoring calculates a numerical relevance value for each document, while ranking orders the documents from highest to lowest score.
- **Word Family:**
  - Noun: Score, Ranking, Scorer
  - Verb: Score, Rank
  - Adjective: Scored, Ranked, Ranking
- **Collocations & Native Pairings:**
  - `verb + term`: *calculate scores, rank documents, compute relevance scores, output ranked lists*
  - `adjective + term`: *lexical score, normalized score, final ranking*
  - `term + noun`: *scoring mechanism, ranking algorithm*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "mark the point for documents" or "grade files"
  - ✅ *Native standard:* "score documents and rank search results based on relevance"
- **Resource & Literature Mapping** *(Conditional: Search systems technical problem)*:
  - **Technical Architecture / Problem:** BM25 scoring formula vs. Dense vector similarity scoring.
  - **Academic Paper / Documentation:** Apache Lucene / Elasticsearch Scoring Guide; *Robertson, S. E., & Zaragoza, H. (2009). The Probabilistic Relevance Framework: BM25 and Beyond.*
  - **GitHub Repo Reference:** `elastic/elasticsearch` (scoring modules).
- **Cross-Domain Bridge (Tech ↔ Economics):** In economics, "ranking" is equivalent to ordinal utility (sorting preferences), while "scoring" is cardinal utility (assigning exact numerical values).

### 2. Optimize
- **IPA:** `/ˈɑːptɪmaɪz/`
- **Definition & Domain Context:** Modifying system code, queries, or model weights to maximize performance (throughput, latency) while minimizing resource consumption (CPU, memory, GPU VRAM).
- **Word Family:**
  - Noun: Optimization, Optimizer
  - Verb: Optimize
  - Adjective: Optimized, Optimal, Sub-optimal
- **Collocations & Native Pairings:**
  - `verb + term`: *optimize performance, run gradient optimization, tune hyperparameters*
  - `adjective + term`: *fully optimized, highly optimal solution, sub-optimal query*
  - `term + noun`: *optimization pipeline, optimizer algorithm (e.g., Adam, SGD)*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "make the code more good" or "unoptimization"
  - ✅ *Native standard:* "optimize the retrieval pipeline for lower latency"
- **Resource & Literature Mapping** *(Conditional: Performance optimization pain point)*:
  - **Technical Architecture / Problem:** Bottlenecks in memory bandwidth during LLM inference.
  - **Academic Paper / Documentation:** PyTorch Optimizer Documentation; *Kingma, D. P., & Ba, J. (2014). Adam: A Method for Stochastic Optimization.*
  - **GitHub Repo Reference:** `pytorch/pytorch` (`torch.optim`).

### 3. Loophole
- **IPA:** `/ˈluːpˌhoʊl/`
- **Definition & Domain Context:** An ambiguity or structural weakness in system rules, business logic, or algorithmic filters that allows users or malicious actors to bypass intended constraints.
- **Word Family:**
  - Noun: Loophole
  - Adjective: Loopholess (rare)
- **Collocations & Native Pairings:**
  - `verb + term`: *exploit a loophole, patch a loophole, find a system loophole*
  - `adjective + term`: *security loophole, algorithmic loophole, design loophole*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "bad hole in the system" or "weak point in code"
  - ✅ *Native standard:* "exploit a loophole in the ranking algorithm by keyword stuffing"
- **Resource & Literature Mapping** *(Conditional: Security and search manipulation vulnerability)*:
  - **Technical Architecture / Problem:** Adversarial search engine optimization (SEO) and prompt injection vulnerabilities.
  - **Academic Paper / Documentation:** OWASP Top 10 API Security Risks.
  - **GitHub Repo Reference:** `OWASP/CheatSheetSeries`.

### 4. Route
- **IPA:** `/ruːt/`
- **Definition & Domain Context:** The operational path through which API requests, network packets, or MQTT messages are directed from a sender to the correct recipient/service.
- **Word Family:**
  - Noun: Route, Routing, Router, Reroute
  - Verb: Route, Reroute
- **Collocations & Native Pairings:**
  - `verb + term`: *route network traffic, configure API routes, handle message routing*
  - `adjective + term`: *dynamic routing, static route, optimized routing path*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "send way for messages"
  - ✅ *Native standard:* "the MQTT broker routes incoming telemetry messages to subscribers"

### 5. Fetch
- **IPA:** `/fetʃ/`
- **Definition & Domain Context:** The programmatic operation of sending a request to a database, API endpoint, or storage system and retrieving the response data.
- **Word Family:**
  - Noun: Fetch, Fetching
  - Verb: Fetch, Re-fetch
- **Collocations & Native Pairings:**
  - `verb + term`: *fetch data asynchronously, execute a fetch request, batch fetch records*
  - `adjective + term`: *asynchronous fetch, remote fetch*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "get data back" or "load data from outside"
  - ✅ *Native standard:* "fetch embedding vectors from the vector database asynchronously"

### 6. Query
- **IPA:** `/ˈkwɪri/`
- **Definition & Domain Context:** A formal request for data or semantic information retrieval sent to a database management system (DBMS) or information retrieval system (IRS).
- **Word Family:**
  - Noun: Query, Querier
  - Verb: Query
  - Adjective: Queryable
- **Collocations & Native Pairings:**
  - `verb + term`: *execute a query, parse a search query, optimize database queries*
  - `adjective + term`: *complex query, natural language query, sparse query*
  - `term + noun`: *query latency, query expansion*
- **Resource & Literature Mapping** *(Conditional: Information retrieval latency problem)*:
  - **Technical Architecture / Problem:** High latency in vector similarity searches over high-dimensional embeddings.
  - **Academic Paper / Documentation:** Pinecone / Milvus Vector Search Architecture Docs.

### 7. Keyword-stuffing
- **IPA:** /ˈkiːwɜːrd ˈstʌfɪŋ/
- **Definition & Domain Context:** An adversarial manipulation tactic in lexical search where candidates or malicious webmasters repeatedly insert high-value keywords to artificially inflate document scores.
- **Word Family:**
  - Noun: Keyword-stuffing
  - Verb: Stuff keywords
- **Collocations & Native Pairings:**
  - `verb + term`: *prevent keyword-stuffing, detect keyword-stuffing exploits*
  - `adjective + term`: *malicious keyword-stuffing*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "spam keyword" or "repeat words many times"
  - ✅ *Native standard:* "candidates can bypass the BM25 filter using keyword-stuffing"
- **Resource & Literature Mapping** *(Conditional: Lexical search vulnerability)*:
  - **Technical Architecture / Problem:** Term Frequency (TF) unbounded scoring vulnerability in BM25.
  - **Academic Paper / Documentation:** Elasticsearch TF-IDF / BM25 Saturation Parameter documentation (`$k_1$`).

### 8. Convert & Transform
- **IPA:** `/kənˈvɜːrt/`, `/trænsˈfɔːrm/`
- **Definition & Domain Context:** Systematically translating raw unstructured data (text, images, audio) into continuous vector representations (embeddings) via neural network encoders.
- **Word Family:**
  - Noun: Conversion, Transformation, Converter, Transformer
  - Verb: Convert, Transform
- **Collocations & Native Pairings:**
  - `verb + term`: *convert text into vectors, transform raw data, apply nonlinear transformations*
  - `adjective + term`: *seamless conversion, linear transformation*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "transmit data into vector"
  - ✅ *Native standard:* "convert raw text into dense embeddings using an encoder model"

### 9. Compensate
- **IPA:** `/ˈkɑːmpənseɪt/`
- **Definition & Domain Context:** Designing system architectures where one component counteracts the inherent weaknesses of another (e.g., combining Lexical BM25 and Dense Retrieval into a Hybrid search model).
- **Word Family:**
  - Noun: Compensation
  - Verb: Compensate
  - Adjective: Compensatory
- **Collocations & Native Pairings:**
  - `verb + term`: *compensate for individual weaknesses, offset trade-offs*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "fix the bad point of the previous thing"
  - ✅ *Native standard:* "Hybrid search was introduced to compensate for the individual weaknesses of BM25 and Dense algorithms."

### 10. Strategy
- **IPA:** `/ˈstrætədʒi/`
- **Definition & Domain Context:** A comprehensive, high-level plan or orientation designed to achieve long-term objectives, distinguishing itself from short-term tactical actions.
- **Word Family:**
  - Noun: Strategy, Strategist
  - Adjective: Strategic, Strategical (rare)
  - Adverb: Strategically
- **Collocations & Native Pairings:**
  - `verb + term`: *formulate a strategy, execute an AI strategy, develop a data strategy*
  - `adjective + term`: *long-term strategy, strategic planning, strategic partnership*
  - `term + noun`: *strategy roadmap, strategy session*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "stategy" (spelling typo) or "plan for everyday work"
  - ✅ *Native standard:* "Formulate a robust AI engineering strategy for system scalability."

### 11. Release
- **IPA:** `/rɪˈliːs/`
- **Definition & Domain Context:** The technical act of deploying code, models, or products from staging to production, or publishing a new version for users.
- **Word Family:**
  - Noun: Release, Releasability
  - Verb: Release
  - Adjective: Releasable, Unreleased
- **Collocations & Native Pairings:**
  - `verb + term`: *release a new model, release code to production, schedule a release*
  - `adjective + term`: *stable release, official release, unreleased version*
  - `term + noun`: *release cycle, release notes, release candidate (RC)*
- **Resource & Literature Mapping** *(Conditional: Software deployment & MLOps pain point)*:
  - **Technical Architecture / Problem:** Zero-downtime deployment pipelines, CI/CD automated release workflows.
  - **Academic Paper / Documentation:** Continuous Delivery (Jez Humble & David Farley); HuggingFace Model Hub Release Guidelines.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "launch code" (Launch implies marketing/PR event, while Release/Deploy is the engineering action)
  - ✅ *Native standard:* "Meta released Llama 3 on HuggingFace and updated the release notes."

### 12. Align & Alignment
- **IPA:** `/əˈlaɪn/`, `/əˈlaɪnmənt/`
- **Definition & Domain Context:** The process of adjusting or fine-tuning AI models, system components, or strategies so that their behavior aligns with human intentions, ethical standards, or business goals.
- **Word Family:**
  - Noun: Alignment, Aligner
  - Verb: Align, Realign
  - Adjective: Aligned, Misaligned
- **Collocations & Native Pairings:**
  - `verb + term`: *align model outputs, ensure model alignment, align technical goals with business strategy*
  - `adjective + term`: *fully aligned, safety-aligned LLM, semantic alignment*
  - `term + noun`: *alignment process, alignment technique (e.g., RLHF, DPO)*
- **Resource & Literature Mapping** *(Conditional: AI Safety and LLM alignment pain point)*:
  - **Technical Architecture / Problem:** Preventing LLMs from generating toxic, harmful, or hallucinated responses.
  - **Academic Paper / Documentation:** *Ouyang et al. (2022). Training language models to follow instructions with human feedback (InstructGPT / RLHF).*
  - **GitHub Repo Reference:** `huggingface/alignment-handbook`.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "make model follow human" or "match the rule together"
  - ✅ *Native standard:* "Align large language models with human preferences using Reinforcement Learning from Human Feedback (RLHF)."

### 13. Spike
- **IPA:** `/spaɪk/`
- **Definition & Domain Context:** In Agile project management, a spike is a time-boxed research or experimentation task designed to resolve technical uncertainties, evaluate new tools, or estimate effort. In systems/DevOps, it refers to a sudden surge in traffic or resource utilization.
- **Word Family:**
  - Noun: Spike
  - Verb: Spike
  - Adjective: Spiky
- **Collocations & Native Pairings:**
  - `verb + term`: *run a technical spike, time-box a spike, handle traffic spikes*
  - `adjective + term`: *technical spike, traffic spike, unexpected CPU spike*
  - `term + noun`: *spike duration, spike task*
- **Resource & Literature Mapping** *(Conditional: Agile estimation uncertainty & system load)*:
  - **Technical Architecture / Problem:** Reducing estimation risk for complex architectural changes or unknown AI model integration limits.
  - **Academic Paper / Documentation:** Scrum Guide / Agile Alliance Technical Spikes Documentation.
  - **GitHub Repo Reference:** Agile project management templates.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "do a test task" or "suddenly jump up high"
  - ✅ *Native standard:* "Allocate a 2-day technical spike to evaluate model inference latency on edge devices."

---

## 📝 Research Journey
- **Why:** To bridge the gap between strong technical comprehension and professional IELTS Academic / native engineering communication, eliminating "Viet-Glish" habits.
- **Struggle:** Translating technical concepts from Vietnamese thoughts often results in imprecise verbs (e.g., *transmit* instead of *convert*, *spam keyword* instead of *keyword-stuffing*).
- **Aha moment:** Realizing that native technical writing relies on precise collocations and established domain terminology (e.g., *saturation function*, *relevance scoring*, *asynchronous fetch*).
- **Career link:** Mastering technical English enables writing clear system documentation, delivering polished engineering presentations, and collaborating globally as an AI Engineer.

## 🔗 Related Topics
- [[personal_evaluations/no1_Q_and_A_English_Evaluation.md]] — initial Q&A evaluation and vocabulary feedback
- [[ai_ml/02-search-systems-lexical-dense-hybrid.md]] — technical foundation for BM25, Dense, and Hybrid search terminology

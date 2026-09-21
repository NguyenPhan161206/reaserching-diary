# Vocabulary & Collocation — Advanced System Architecture & AI Terminology

**Created Date:** 2026-09-21
**Last Updated:** 2026-09-21
**Tags:** `#vocabulary`, `#it`, `#ai_ml`, `#system-design`
**Status:** Published

---

## 📌 Term List & Quick Reference

| Term / Phrase | IPA | Core Meaning | Domain |
|---------------|-----|--------------|--------|
| **Latency** | `/ˈleɪtnsi/` | The delay before a transfer of data begins following an instruction | System Design |
| **Idempotency** | `/ˌaɪdəmˈpoʊtnsi/` | Property where an operation can be applied multiple times without changing the result beyond the initial application | Software Engineering / APIs |
| **Orchestration** | `/ˌɔːrkɪˈstreɪʃn/` | Automated configuration, coordination, and management of computer systems and software | DevOps / Microservices |
| **Inference** | `/ˈɪnfərəns/` | The process of running live data through a trained AI model to make a prediction | AI / MLOps |
| **Hallucination** | `/həˌluːsɪˈneɪʃn/` | When an AI model confidently generates false or illogical information | AI / LLMs |
| **Embedding** | `/ɪmˈbedɪŋ/` | A representation of data (like text) as a continuous vector in a low-dimensional space | AI / Search Systems |
| **Scalability** | `/ˌskeɪləˈbɪləti/` | The measure of a system's ability to increase or decrease in performance and cost in response to changes in application and system processing demands | Cloud / System Architecture |
| **Resilience** | `/rɪˈzɪliəns/` | The ability of a system to recover from failures and continue to function | System Design |
| **Data Pipeline** | `/ˈdeɪtə ˈpaɪplaɪn/` | A set of actions that extract data (or analytics and insights) from various sources | Data Engineering |
| **Fine-tuning** | `/ˌfaɪn ˈtuːnɪŋ/` | The process of taking a pre-trained model and training it further on domain-specific data | AI / MLOps |

---

## 🧠 Detailed Vocabulary Analysis

### 1. Latency
- **IPA:** `/ˈleɪtnsi/`
- **Definition & Domain Context:** In system design, latency is the time it takes for a data packet to travel from a source to a destination. In AI inference, it is the time taken to generate a response.
- **Word Family:**
  - Noun: Latency
  - Adjective: Latent
- **Collocations & Native Pairings:**
  - `adjective + term`: *low latency, high latency, network latency, ultra-low latency*
  - `verb + term`: *reduce latency, minimize latency, experience latency*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "the time is slow"
  - ✅ *Native standard:* "optimize the system to achieve ultra-low network latency"
- **Resource & Literature Mapping:**
  - **Technical Architecture / Problem:** Reducing inference time in real-time AI applications.
- **Cross-Domain Bridge (Tech ↔ Economics):** In economics, "latency" can refer to the time delay between a policy implementation and its observable effect on the economy.

### 2. Idempotency
- **IPA:** `/ˌaɪdəmˈpoʊtnsi/`
- **Definition & Domain Context:** In API design, an idempotent operation can be called multiple times with the same input, producing the same result without unintended side effects (e.g., charging a customer twice).
- **Word Family:**
  - Noun: Idempotency, Idempotence
  - Adjective: Idempotent
- **Collocations & Native Pairings:**
  - `adjective + term`: *idempotent API, idempotent operation, idempotent request*
  - `verb + term`: *ensure idempotency, implement idempotency keys*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "make sure the API doesn't duplicate"
  - ✅ *Native standard:* "implement idempotency keys to safely retry failed API requests"
- **Resource & Literature Mapping:**
  - **Technical Architecture / Problem:** Payment gateways and distributed systems where network failures require safe retries.

### 3. Orchestration
- **IPA:** `/ˌɔːrkɪˈstreɪʃn/`
- **Definition & Domain Context:** The automated management of complex computer systems, middleware, and services, often associated with containers (e.g., Kubernetes).
- **Word Family:**
  - Noun: Orchestration, Orchestrator
  - Verb: Orchestrate
- **Collocations & Native Pairings:**
  - `noun + term`: *container orchestration, workflow orchestration*
  - `verb + term`: *orchestrate microservices, orchestrate deployments*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "manage many servers together"
  - ✅ *Native standard:* "leverage Kubernetes for container orchestration and automated scaling"

### 4. Inference
- **IPA:** `/ˈɪnfərəns/`
- **Definition & Domain Context:** The deployment phase of machine learning where a trained model is used to predict outputs for new, unseen data.
- **Word Family:**
  - Noun: Inference
  - Verb: Infer
  - Adjective: Inferential
- **Collocations & Native Pairings:**
  - `noun + term`: *inference engine, inference time, batch inference, real-time inference*
  - `verb + term`: *run inference, deploy for inference*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "run the model to get result"
  - ✅ *Native standard:* "deploy the LLM on GPUs to accelerate real-time inference"

### 5. Hallucination
- **IPA:** `/həˌluːsɪˈneɪʃn/`
- **Definition & Domain Context:** A phenomenon in generative AI where a model outputs plausible but factually incorrect or nonsensical information.
- **Word Family:**
  - Noun: Hallucination
  - Verb: Hallucinate
- **Collocations & Native Pairings:**
  - `verb + term`: *reduce hallucinations, mitigate hallucinations, prone to hallucination*
  - `adjective + term`: *AI hallucination, factual hallucination*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "the AI says wrong things"
  - ✅ *Native standard:* "implement RAG (Retrieval-Augmented Generation) to ground the model and mitigate hallucinations"

### 6. Embedding
- **IPA:** `/ɪmˈbedɪŋ/`
- **Definition & Domain Context:** A machine learning technique that converts categorical data, like text or images, into continuous numerical vectors so that similar items are close in the vector space.
- **Word Family:**
  - Noun: Embedding
  - Verb: Embed
- **Collocations & Native Pairings:**
  - `noun + term`: *word embedding, vector embedding, embedding space, embedding model*
  - `verb + term`: *generate embeddings, compute embeddings*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "change text to numbers"
  - ✅ *Native standard:* "generate dense vector embeddings for semantic search"

### 7. Scalability
- **IPA:** `/ˌskeɪləˈbɪləti/`
- **Definition & Domain Context:** A system's capacity to handle growing amounts of work by adding resources (horizontal scaling) or upgrading existing resources (vertical scaling).
- **Word Family:**
  - Noun: Scalability
  - Verb: Scale
  - Adjective: Scalable
- **Collocations & Native Pairings:**
  - `adjective + term`: *high scalability, horizontal scalability, vertical scalability*
  - `verb + term`: *ensure scalability, design for scalability*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "make system support more users"
  - ✅ *Native standard:* "architect the microservices to ensure horizontal scalability under heavy load"

### 8. Resilience
- **IPA:** `/rɪˈzɪliəns/`
- **Definition & Domain Context:** System resilience is the ability to gracefully handle and recover from partial failures (e.g., a database crash) without bringing down the entire application.
- **Word Family:**
  - Noun: Resilience
  - Adjective: Resilient
- **Collocations & Native Pairings:**
  - `adjective + term`: *system resilience, operational resilience*
  - `verb + term`: *build resilience, improve resilience, test resilience*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "make system not die"
  - ✅ *Native standard:* "build fault tolerance and resilience into the distributed architecture"
- **Cross-Domain Bridge (Tech ↔ Economics):** Economic resilience refers to an economy's ability to recover from macro-shocks (like a recession or pandemic).

### 9. Data Pipeline
- **IPA:** `/ˈdeɪtə ˈpaɪplaɪn/`
- **Definition & Domain Context:** An automated set of processes that extract data from various sources, transform it into a usable format, and load it into a data warehouse or data lake (ETL/ELT).
- **Word Family:**
  - Noun: Pipeline
  - Verb: Pipelining
- **Collocations & Native Pairings:**
  - `adjective + term`: *robust data pipeline, real-time data pipeline, automated pipeline*
  - `verb + term`: *build a data pipeline, orchestrate a data pipeline, ingest via pipeline*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "code to move data"
  - ✅ *Native standard:* "build a robust ETL data pipeline to continuously ingest telemetry data"

### 10. Fine-tuning
- **IPA:** `/ˌfaɪn ˈtuːnɪŋ/`
- **Definition & Domain Context:** A transfer learning approach where a pre-trained foundational model is further trained on a smaller, domain-specific dataset to adapt it to a specialized task.
- **Word Family:**
  - Noun: Fine-tuning
  - Verb: Fine-tune
  - Adjective: Fine-tuned
- **Collocations & Native Pairings:**
  - `noun + term`: *instruction fine-tuning, parameter-efficient fine-tuning (PEFT)*
  - `verb + term`: *fine-tune a model, require fine-tuning*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "train the model again for my data"
  - ✅ *Native standard:* "fine-tune the foundational LLM on proprietary legal documents to improve domain accuracy"

---

## 📝 Research Journey
- **Why:** To expand technical communication capabilities in AI engineering and system architecture.
- **Struggle:** Differentiating between nuanced system design terms (e.g., Latency vs Throughput).
- **Aha moment:** Realizing that these architectural concepts apply to both software and business operations.
- **Career link:** Essential for system design interviews and writing technical RFCs as an AI Engineer.

## 🔗 Related Topics
- [[01-core-technical-collocations-and-viet-glish-fixes.md]] — core engineering collocations
- [[02-agile-iot-project-management-terminology.md]] — agile project management

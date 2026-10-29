# Scientific Research Methodology — Comprehensive Guide & Epistemological Framework

**Created Date:** 2026-10-21
**Last Updated:** 2026-10-21
**Tags:** `#ResearchMethodology`, `#Epistemology`, `#Science`, `#AIResearch`, `#EmpiricalMethods`
**References:** [Kuhn (1962), Popper (1959), Creswell & Creswell (2017), Wohlin et al. (2012)]
**Status:** Published
**Difficulty:** Advanced

---

## 📌 Key Takeaways (TL;DR)
- **Empirical Rigor:** Science is not merely gathering facts; it is the systematic testing of falsifiable hypotheses against observable, measurable evidence.
- **Validity as Core Principle:** Research design stands or falls on four validity pillars: *Internal* (causality), *External* (generalizability), *Construct* (measurement accuracy), and *Statistical Conclusion* (inference strength).
- **AI/ML as Empirical Science:** Developing and evaluating machine learning models is fundamentally an applied scientific process; benchmark overfitting, data leakage, and lack of ablation studies are methodological failures, not merely engineering bugs.

---

## 🧠 Detailed Analysis

### 1. Epistemological Foundations & Philosophy of Science
Before conducting research, one must understand *how knowledge is acquired and validated*.

#### Major Epistemological Frameworks
- **Positivism:** Assumes an objective reality exists independent of the observer. Knowledge is verified through direct observation and quantitative measurement. (Dominant in hard sciences and traditional ML benchmarks).
- **Post-Positivism:** Acknowledges that absolute objective truth is unattainable due to human cognitive limits and measurement error. Emphasizes *probability*, falsification, and triangulation across multiple methods.
- **Constructivism / Interpretivism:** Assumes reality is socially constructed and subjective. Emphasizes qualitative understanding of human contexts, motivations, and interactions (relevant to Human-AI Interaction and HCI).

#### Karl Popper and Falsifiability
- **Principle:** A theory cannot be conclusively proven true by finite observations (the problem of induction), but it can be proven false by a single contradictory observation.
- **Application in AI:** Claiming "our neural network model is 100% robust" is unscientific. A scientific claim must specify the conditions under which it will fail (e.g., adversarial perturbation magnitude $\epsilon > 0.03$).

#### Thomas Kuhn and Paradigm Shifts
- **Normal Science:** Research conducted within an accepted theoretical framework (paradigm).
- **Crisis & Revolution:** Accumulation of anomalies that the current paradigm cannot explain leads to a scientific crisis and a paradigm shift (e.g., transition from symbolic AI to connectionist deep learning, and later to transformer-based foundation models).

---

### 2. The Complete Research Pipeline (6 Phases)

```
[1. Problem Formulation] ──> [2. Literature Review] ──> [3. Research Design]
                                                               │
[6. Peer Review & Report]  <── [5. Data Analysis]    <── [4. Data Collection]
```

#### Phase 1: Problem Formulation & Research Question Design
- **Identification:** Spotting gaps, contradictions, or inefficiencies in existing systems.
- **Criteria (FINER):** Feasible, Interesting, Novel, Ethical, Relevant.
- **Hypothesis Formulation:** 
  - Null Hypothesis ($H_0$): Assumes no significant effect, difference, or relationship.
  - Alternative Hypothesis ($H_1$): Assumes a statistically significant effect or relationship.

#### Phase 2: Literature Review & Synthesis
- **Systematic Literature Review (SLR):** Using explicit, reproducible criteria (e.g., PRISMA guidelines) to identify, appraise, and synthesize all empirical evidence addressing a specific research question.
- **Taxonomy Mapping:** Organizing prior literature by methodology, dataset, performance metric, and identified limitations.

#### Phase 3: Research Design & The Validity Framework
A rigorous research design must defend against four major threats to validity (Cook & Campbell):
1. **Internal Validity:** Can we establish a causal relationship rather than mere correlation? (Threats: confounding variables, selection bias, history effects).
2. **External Validity:** Can the findings generalize across different populations, datasets, hardware environments, and domains?
3. **Construct Validity:** Do our operational metrics actually measure the theoretical concepts they claim to? (e.g., Does BLEU score actually measure translation "quality" or just surface n-gram overlap?).
4. **Statistical Conclusion Validity:** Are our statistical tests adequately powered, and are assumptions met? (Threats: p-hacking, low sample size, multiple testing errors).

#### Phase 4: Data Collection & Measurement
- **Sampling Techniques:** Random sampling, stratified sampling, convenience sampling, and quota sampling.
- **Instrumentation:** Designing surveys, test suites, or telemetry sensors with minimal measurement error.
- **Data Hygiene:** Guarding against data contamination, survivorship bias, and missing data imputation artifacts.

#### Phase 5: Quantitative & Qualitative Analysis
- **Quantitative Analysis:** 
  - Descriptive statistics (mean, median, variance, interquartile range).
  - Inferential statistics (hypothesis testing, ANOVA, regression analysis, Bayesian inference).
  - Effect size calculation (Cohen’s $d$, odds ratios) rather than relying solely on p-values ($p < 0.05$).
- **Qualitative Analysis:** 
  - Thematic analysis, open/axial coding, discourse analysis, and triangulation.

#### Phase 6: Interpretation, Limitations & Peer Review
- Discussing findings in relation to prior literature.
- Transparently acknowledging limitations and threats to validity.
- Subjecting the manuscript or system artifact to rigorous double-blind peer review.

---

### 3. Application to Computer Science & AI Engineering

In modern computer science and AI, the scientific method is operationalized through **Empirical Software Engineering (ESE)** and **Rigorous ML Evaluation**:

| Research Element | Traditional Science | AI & Software Engineering Equivalent |
|------------------|---------------------|---------------------------------------|
| **Theory** | Physical or social law | Architectural hypothesis (e.g., self-attention improves long-range dependency capture) |
| **Experiment** | Laboratory trial | Benchmark evaluation run on standardized datasets (ImageNet, MMLU) |
| **Control Group** | Placebo / control cohort | Baseline model (e.g., standard LSTM vs. Transformer) |
| **Ablation** | Removing a biological organ | Removing a architectural component (e.g., feed-forward network, skip connection) |
| **Replication** | Independent lab rerun | Re-implementing paper from scratch and matching reported metrics |

#### Common Methodological Pitfalls in AI Research
1. **Data Leakage:** Information from the test set inadvertently contaminating the training or validation phase (e.g., normalization computed over the entire dataset before splitting).
2. **Over-fitting to Benchmarks:** Tuning hyperparameters repeatedly against the public test set, effectively invalidating its independence.
3. **Lack of Ablation Studies:** Claiming a complex system works without isolating *which* specific component drives the performance gain.
4. **Seed Sensitivity:** Reporting results from a single random seed that happened to yield favorable metrics rather than reporting mean and variance across multiple seeds.

---

### 4. Research Ethics & Scientific Integrity
- **Human Subjects & IRB:** Ensuring informed consent, data privacy, and minimal harm when conducting user studies or collecting telemetry.
- **Data Integrity:** Prohibition against data fabrication, selective reporting (p-hacking), and HARKing (Hypothesizing After Results are Known).
- **Attribution & Plagiarism:** Proper citation of prior work, open-source licenses, and reproducible code repositories.
- **Responsible AI:** Evaluating models for fairness, bias, disparate impact, and environmental cost (carbon footprint of training large models).

---

## 📝 Research Journey
- **Why:** As an aspiring AI Engineer, building models without understanding research methodology leads to trial-and-error superstition rather than principled engineering. Understanding how to frame hypotheses and measure performance objectively bridges raw coding with scientific discovery.
- **Struggle:** Initially viewing research as just "writing papers" rather than a structured framework for eliminating cognitive bias and uncertainty.
- **Aha moment:** Realizing that an ablation study in deep learning is mathematically identical to a controlled experiment with isolation of variables in physics.
- **Career link:** Empowers me to evaluate academic papers critically, design robust internal benchmarks for production AI systems, and avoid common data leakage and evaluation pitfalls.

---

## 🔗 Related Topics
- [[books_summaries/01-interdisciplinary-thinking-analysis.md]] — Systems thinking and mental models across disciplines.
- [[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]] — Operationalizing empirical model evaluation in production pipelines.
- [[books_summaries/06-building-a-second-brain/01-para-and-progressive-summarization.md]] — Managing research literature using PARA and progressive summarization.
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — Algorithmic problem-solving and experimental benchmarking.
- [SUGGESTED] `books_summaries/08-statistical-methods-for-machine-learning.md` — Deep dive into hypothesis testing, confidence intervals, and p-values for ML evaluation.
- [SUGGESTED] `ai_ml/09-reproducibility-and-evaluation-pitfalls.md` — Case studies of famous AI paper retractions and methodological flaws.

---

## 🤔 Open Questions
- [ ] How can automated agentic systems (like LLM researchers) be designed to adhere strictly to falsifiability and prevent hallucinated hypotheses?
- [ ] What is the optimal balance between exploratory rapid prototyping and formal empirical research methodology in fast-paced software start-ups?

---
*Science is built of facts, as a house is of stones. But a collection of facts is no more a science than a heap of stones is a house. — Henri Poincaré*

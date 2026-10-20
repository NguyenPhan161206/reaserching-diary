# Vocabulary & Collocation — Fraud Detection Reading

**Created Date:** 2026-10-21
**Last Updated:** 2026-10-21
**Tags:** `#vocabulary`, `#collocations`, `#technical-english`, `#fraud-detection`, `#evaluation`
**Status:** Active
**Template:** Template V — Advanced Vocabulary & Collocation Note
**Source context:** nine vocabulary items requested during a fraud detection learning conversation. English explanations are canonical here; Vietnamese below appears only as marked source-language quotations or translation glosses.
**Code repository:** [NguyenPhan161206/Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection) · `git@github.com:NguyenPhan161206/Credit_Card_Fraud_Detection.git`

## 📌 Term List & Quick Reference

IPA uses American English forms; pronunciation varies by accent.

| Term / Phrase | IPA | Core Meaning | Domain |
| --- | --- | --- | --- |
| indicate | /ˈɪndɪkeɪt/ | show or suggest; not necessarily prove | Evidence, interpretation |
| transaction | /trænˈzækʃən/ | an exchange or business operation, such as a payment | Finance, databases |
| trigger | /ˈtrɪɡər/ | cause an event or action to begin | Events, alerts |
| process | /ˈprɑːses/ | a series of actions; also to handle or transform something | General, computing |
| investigation | /ɪnˌvestɪˈɡeɪʃən/ | systematic examination to establish facts | Fraud, incident analysis |
| precision | /prɪˈsɪʒən/ | in classification, the fraction of predicted positives that are correct | ML evaluation |
| recall | /ˈriːkɔːl/ (noun) | in classification, the fraction of actual positives detected | ML evaluation |
| assume | /əˈsuːm/ | accept a premise for reasoning, without establishing it as fact | Reasoning, modeling |
| proportion | /prəˈpɔːrʃən/ | a part relative to the whole | Statistics, reporting |

Spelling corrections from the lookups: `transtraction` → **transaction**;
`presision` → **precision**; `propotion` → **proportion**.

## 🧠 Detailed Vocabulary Analysis

### 1. Indicate

- **Definition & Domain Context:** To show, point to, or suggest something. In a scientific sentence, specify what the evidence supports. *Indicate* does not by itself establish certainty or causality. Translation gloss: “cho thấy; biểu thị; báo hiệu”.
- **Word Family:** verb *indicate*; nouns *indication*, *indicator*; adjective *indicative*. *Indicative of* is common; no everyday adverb is needed here.
- **Collocations & Native Pairings:** *results indicate that…*, *indicate a risk*, *an indicator of fraud*, *be indicative of a problem*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Avoid *“The score indicates about fraud.”* Use *“The score indicates elevated fraud risk.”* Avoid turning *indicates* into *proves* when the evidence is uncertain.
- **Example:** *“Higher scores indicate stronger evidence of fraud, assuming that is how this model's scores are defined.”*
- **Cross-Domain Bridge:** A business indicator, like a model score, informs a decision but does not automatically establish its cause. This distinction prevents overconfident claims.

### 2. Transaction

- **Definition & Domain Context:** In the present finance context, a recorded exchange such as a card payment. In databases, a transaction is a unit of database work; that is a related but distinct technical meaning. Translation gloss: “giao dịch”. One payment can involve several database transactions or records; do not assume they map one-to-one.
- **Word Family:** noun *transaction*; verb *transact*; adjective *transactional*; adverb *transactionally* (technical).
- **Collocations & Native Pairings:** *process a transaction*, *authorize a payment*, *flag a suspicious transaction*, *transaction amount*, *transaction history*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Correct the spelling *transtraction*. Prefer *“The system processes card transactions”* to *“The system does card transactions”* when describing its function.
- **Example:** *“One row represents a transaction”* is a statement to verify from the dataset documentation, not a universal rule.
- **Cross-Domain Bridge:** Payment events connect customer activity to the database records engineers analyze; knowing the unit of observation prevents double counting.

### 3. Trigger

- **Definition & Domain Context:** As a verb, cause an event or action to start; as a noun, the condition or event that starts it. Translation gloss: “kích hoạt; làm phát sinh”. An alert trigger is not proof of wrongdoing.
- **Word Family:** noun/verb *trigger*; participles *triggered*, *triggering*; compound adjective *event-triggered*.
- **Collocations & Native Pairings:** *trigger an alert*, *trigger an investigation*, *be triggered by*, *a triggering condition*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Use *“The transaction triggers an alert”*, not *“The transaction trigger an alert”*: a singular present-tense subject takes *triggers*.
- **Example:** *“Crossing the threshold triggers an alert.”* The threshold condition starts the alerting action; a separate investigation may follow.
- **Cross-Domain Bridge:** Event-driven software starts actions when conditions occur; operational teams must decide what action an alert should initiate.

### 4. Process

- **Definition & Domain Context:** As a noun, a series of actions or changes; as a verb, to handle or transform input. Translation gloss for the requested source word: “quá trình”. The existing [process/procedure/pipeline/lifecycle note](16-process-procedure-pipeline-lifecycle.md) contains the broader distinctions; this entry anchors the current reading context.
- **Word Family:** noun/verb *process*; noun *processing*; participle *processed*. *Processor* names an agent/device in appropriate contexts.
- **Collocations & Native Pairings:** *the training process*, *process a transaction*, *data processing*, *during the investigation*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** *Process* and *procedure* are not automatic substitutes: a process describes actions/change; a procedure specifies how to carry out a task. *Learning process* is valid when discussing mechanisms of learning; *learning journey* emphasizes personal experience.
- **Example:** *“The training process includes fitting the model on training data.”*
- **Cross-Domain Bridge:** A technical process and a business process both involve actions, but their boundaries and intended outcomes must be defined before measuring performance.

### 5. Investigation

- **Definition & Domain Context:** A careful, systematic examination to establish facts or determine what happened. Translation gloss: “sự điều tra; quá trình điều tra”. An investigation is an activity; its findings are the resulting conclusions or evidence.
- **Word Family:** verb *investigate*; nouns *investigation*, *investigator*; adjective *investigative*. No common adverb is necessary for this context.
- **Collocations & Native Pairings:** *conduct an investigation*, *launch an investigation*, *investigate a transaction*, *under investigation*, *investigation findings*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Use *“investigate the transaction”*, not *“investigate about the transaction”*. With the noun, *“an investigation into suspected fraud”* is natural.
- **Example:** *“The investigation outcome became available after the transaction.”* If true for a dataset, it matters for what can be used at prediction time.
- **Cross-Domain Bridge:** Fraud review and software incident investigation both distinguish an initial signal from a subsequently established explanation.

### 6. Precision

- **Definition & Domain Context:** In binary classification with fraud as the positive class, the fraction of predicted fraud alerts that correspond to actual fraud. Translation gloss: “tỷ lệ cảnh báo đúng trong tổng số cảnh báo”. General English *precision* can mean exactness; floating-point precision is a different computing sense.
- **Word Family:** noun *precision*; adjective *precise*; adverb *precisely*. There is no ordinary verb *precision*; use *measure* or *specify precisely* as appropriate.
- **Collocations & Native Pairings:** *high precision*, *precision at a chosen threshold*, *report precision and recall*. A precise measurement is not the same statement as a classifier having high precision.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Correct *presision* to *precision*. Avoid calling precision *overall accuracy*: their denominators differ.
- **Example:** 8 correct fraud alerts out of 20 alerts gives $8/20=40\%$ precision. These counts are hypothetical, not a dataset result.
- **Cross-Domain Bridge:** Lower precision means a larger fraction of alerts require review despite being false alarms, affecting investigation workload.

### 7. Recall

- **Definition & Domain Context:** In the same classification setting, the fraction of actual fraud cases correctly detected. Translation gloss: “tỷ lệ phát hiện được gian lận”. Outside ML, *recall* can mean remembering something or withdrawing a product; neither is the metric's definition.
- **Word Family:** noun/verb *recall*; *recalled*, *recalling*; adjective *recallable* is uncommon and unnecessary for naming this metric. The verb often has second-syllable stress in American English.
- **Collocations & Native Pairings:** *high recall*, *improve recall*, *recall on the positive class*, *precision and recall*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Avoid *“Recall is the percentage of correct alerts.”* Use *“Recall is the percentage of actual fraud cases detected.”* Correct alerts among all alerts describes precision.
- **Example:** Detecting 8 of 10 actual fraud cases gives $8/10=80\%$ recall. This is the same hypothetical case as the precision example.
- **Cross-Domain Bridge:** Recall describes coverage of the events a team wants to catch; missed cases and false alarms create different consequences.

### 8. Assume

- **Definition & Domain Context:** Accept a premise as true for the purpose of reasoning without claiming it has been established. Translation gloss: “giả sử; giả định; cho rằng”. Assumptions can be reasonable or well-supported; calling something an assumption does not make it arbitrary.
- **Word Family:** verb *assume*; noun *assumption*; adverb *presumably* expresses a related idea but is not the same word family. Use *assumed* as a participial adjective.
- **Collocations & Native Pairings:** *assume that…*, *assume a value*, *under this assumption*, *make an assumption*, *check an assumption*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Write *“We assume that labels are correct”* to state a premise, not *“Labels are correct”* when that has not been verified. *Assume responsibility* has another sense: take responsibility.
- **Example:** *“Assume higher scores represent greater fraud risk.”* This establishes the score direction for the explanation, not a fact about every model.
- **Cross-Domain Bridge:** Models and financial decisions both depend on assumptions; stating them makes it possible to examine when conclusions would change.

### 9. Proportion

- **Definition & Domain Context:** A part relative to its whole, usually expressed between 0 and 1 or as a percentage. Translation gloss: “tỷ lệ; phần trong tổng thể”. A general ratio compares two quantities, which need not be part and whole.
- **Word Family:** noun *proportion*; adjective *proportional*; adverb *proportionally*; adjective *proportionate* and adverb *proportionately* are also used. The verb *proportion* exists but is uncommon in this reading context.
- **Collocations & Native Pairings:** *the proportion of fraudulent transactions*, *a large proportion of alerts*, *express a proportion as a percentage*.
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:** Correct *propotion* to *proportion*. Prefer *“The proportion is 0.02, or 2%”* to *“The proportion is 2”* when expressing a two-percent share.
- **Example:** 2 fraudulent transactions in a hypothetical total of 100 yield $2/100=0.02=2\%$.
- **Cross-Domain Bridge:** Fraud prevalence, market share and alert precision are all proportions, but their denominators answer different questions. Always state the whole.

## Resource & Literature Mapping

The metric definitions above follow [scikit-learn's official precision/recall documentation](https://scikit-learn.org/stable/modules/model_evaluation.html#precision-recall-f-measure-metrics) (consulted 2026-10-21). For fraud as the positive class:

$$
\mathrm{Precision}=\frac{TP}{TP+FP},\qquad
\mathrm{Recall}=\frac{TP}{TP+FN}.
$$

$TP$ is detected fraud, $FP$ is a false fraud alert, and $FN$ is missed fraud.
When a denominator is zero, its corresponding ratio is mathematically undefined;
software may report a configured value, so state the convention. Do not silently
compare metrics with different positive-class definitions or averaging rules.

The local [fraud detection foundations note](../ai_ml/08-fraud-detection-foundations.md)
provides the conceptual setting. The linked code repository is the location for
future executable investigations; no dataset experiment is claimed here.

## 📝 Research Journey

- **Why:** The user requested these translations while reading fraud detection explanations, then explicitly asked to preserve them in this diary.
- **Struggle:** Recorded spelling questions were *transtraction*, *presision* and *propotion*. Other personal difficulties have not been reported and should not be invented.
- **Aha moment:** Not yet supplied by the learner. The note documents distinctions without claiming mastery or a personal breakthrough.
- **Career link:** Distinguishing evidence, assumptions, alerting actions and metric denominators supports clear communication of ML findings in an AI engineering context.

## 🔗 Related Topics

- [Fraud detection foundations](../ai_ml/08-fraud-detection-foundations.md) — the concepts behind the source sentences.
- [Process, procedure, pipeline and lifecycle](16-process-procedure-pipeline-lifecycle.md) — the full source-word distinction for the marked quotation “quá trình”.
- [Document and communication verbs](06-document-and-communication-verb-family.md) — *precise*, *accurate* and the register of evidence claims.
- [Labelling and temporary memory](07-labelling-and-temporary-memory.md) — labels and the previously suggested evaluation vocabulary.
- [Opportunity cost](../economics_business/02-opportunity-cost.md) — the allocation of limited review effort; it does not itself determine an optimal alert threshold.
- 🔗 [Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection) — code & explanations live there; this diary holds concept anchors and roadmap.
- [SUGGESTED] **Evidence-strength verbs** (*suggest, indicate, support, establish, demonstrate*) — useful for distinguishing tentative interpretations from stronger claims; fits `vocabulary/`.
- [SUGGESTED] **Decision and uncertainty vocabulary** (*threshold, score, likelihood, probability, calibration*) — useful because a numerical score is not automatically a calibrated probability; fits `vocabulary/`.

No exercises or practice activities are added, in accordance with the user's request.

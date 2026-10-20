# Fraud Detection Foundations: From Recorded Labels to Decision Boundaries

**Created Date:** 2026-10-21
**Last Updated:** 2026-10-21
**Tags:** `#fraud-detection`, `#classification`, `#label-quality`, `#evaluation`, `#precision`, `#recall`, `#decision-boundary`
**References:** Learning conversation on 2026-10-21; primary documentation linked below; source repositories listed in Section 6.
**Status:** Draft — consolidated learning reference; understanding and dataset-specific claims remain unverified.
**Difficulty:** Beginner to Intermediate
**Code repository:** [NguyenPhan161206/Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection) · `git@github.com:NguyenPhan161206/Credit_Card_Fraud_Detection.git`

## 📌 Key Takeaways (TL;DR)

- Features describe a transaction; the target specifies what we want to predict. Many features can still lead to a binary decision.
- A recorded label can differ from reality. Label delay and incorrect labels affect both training and evaluation.
- An alert is a decision to investigate, not confirmation of fraud. A score is not automatically a calibrated probability.
- Precision measures the fraction of alerts that are correct. Recall measures the fraction of recorded fraud that is detected.
- The confusion matrix compares decisions with outcomes, not one feature with another.
- A decision threshold converts scores into labels and determines the alert region. Precision need not change monotonically with threshold.
- All numerical transactions and results below are hypothetical illustrations, not measured findings from either project dataset.

## 🧠 Detailed Notes

### 1. Core Concepts

#### 1.1 Observation, features, target and prediction

One observation is usually one transaction. A feature vector describes it:

$$
x_i=(x_{i1},x_{i2},\ldots,x_{id}),\qquad X\in\mathbb{R}^{n\times d}.
$$

Here $n$ is the number of observations and $d$ is the number of numeric input
features after any required encoding. The target $y_i$ records the outcome;
for our binary examples, $1$ denotes fraud and $0$ denotes legitimate.
The prediction $\hat y_i$ is the model's decision, not the actual outcome.

| Transaction | Amount | Hour | Distance from usual location | Recorded target |
|---|---:|---:|---:|---:|
| A | $20 | 10 | 2 km | 0 |
| B | $900 | 3 | 800 km | 1 |
| C | $500 | 14 | 5 km | 0 |

These illustrative columns are not a verified schema of the project datasets.
A transaction ID may identify a row without being an appropriate predictor.
The target must not be included among input features.

**Why this matters:** a data table describes observations. Evaluation describes
how decisions compare with outcomes. Confusing these purposes leads to the
mistaken idea that each feature needs its own row in the confusion matrix.

#### 1.2 Real outcome versus recorded label

A recorded label is the conclusion available to the collection process. It is
not an unconditional guarantee of the real outcome. Consider a hypothetical
stolen-card purchase of $200:

| Time | Event | Real outcome | Recorded label in this example |
|---|---|---|---|
| Day 1 | Transaction occurs, no report yet | Fraud | 0 |
| Day 5 | Cardholder reports unauthorized use | Fraud | Awaiting confirmation |
| Day 8 | Investigation confirms fraud | Fraud | 1 |

The event did not become fraudulent on Day 8; knowledge about it changed.
An export made on Day 2 could retain the initial label. This is **label delay**.
If the retained label disagrees with reality, it is also **label noise**.
Delay does not necessarily create incorrect labels in a released dataset:
its provider may wait for outcomes to mature before exporting it.

During supervised training, a prediction is compared with the recorded target.
A high fraud score for an actual fraud labeled $0$ is treated as an error by
an objective that rewards agreement with those labels. Updates can therefore
push predictions in the wrong direction. The impact depends on the amount,
pattern and learning method; one wrong label does not determine an entire model.
Systematic errors, such as one fraud subtype repeatedly going unrecognized,
can create a persistent blind spot.

Evaluation has the same dependency. If the test label is incorrectly $0$:

| Model decision | Evaluation against the incorrect label | Interpretation if the real outcome is fraud |
|---|---|---|
| Alert | False positive | Correct detection |
| No alert | True negative | Missed fraud |

**Why this matters:** a faulty answer key can teach the wrong lesson and grade
an actually correct answer as wrong. For each dataset, investigate label origin,
confirmation delay, investigated population, updates and the meaning of $0$.
Does it mean confirmed legitimate, or no fraud recorded by the collection date?
An unanswered question is **unknown**, not evidence that the dataset is defective.

#### 1.3 Class imbalance and the accuracy trap

Suppose 10,000 transactions contain 9,990 legitimate transactions and 10 fraud
cases. Predicting legitimate for every transaction gives:

$$
\text{Accuracy}=\frac{TP+TN}{TP+TN+FP+FN}=\frac{9990}{10000}=99.9\%.
$$

Yet the system detects zero fraud. **Class imbalance** means one class is much
more frequent than another. Accuracy is not inherently invalid, but here it
conceals complete failure to detect the minority class.

**Why this matters:** the metric must answer the intended question. A high
accuracy alone does not establish useful fraud detection.

#### 1.4 Data leakage and prediction-time availability

A feature must be available at the intended prediction time. A later investigation
outcome cannot be used to predict fraud when the transaction first arrives.
That would give the model future information: **data leakage**.
Leakage can also occur when preprocessing learns from held-out data.

**Why this matters:** apparent performance can depend on information unavailable
in actual use. For every feature, distinguish event time, availability time and
label-confirmation time. Keep train, validation and test roles separate; fit
learned preprocessing only on the relevant training partition.

### 2. How It Works (Mechanism)

#### 2.1 The full prediction and evaluation chain

```text
Available transaction features
             |
             v
      Model scoring rule f(X)
             |
             v
          Score s
             |
       compare with t
             |
             v
    Predicted class / alert
             |
  compare with recorded outcome
             |
             v
      Confusion matrix
             |
             v
    Precision and recall
```

For the examples here, higher scores indicate stronger evidence of fraud:

$$
s_i=f(x_i),\qquad \hat y_i=\mathbf{1}[s_i\geq t].
$$

A score, an estimated probability and an operational alert are different objects.
A score of $0.8$ need not imply an 80% fraud probability. Calibration concerns
whether stated probabilities agree with observed event frequencies across
appropriate groups of cases; it is not established just by a score lying in $[0,1]$.

#### 2.2 Confusion matrix: decisions versus outcomes

Assume reliable recorded outcomes for this worked example: 20 alerts, 8 fraud
cases among those alerts and 10 fraud cases in total.

| System decision | Recorded fraud: 1 | Recorded legitimate: 0 |
|---|---:|---:|
| Alert: 1 | TP = 8 | FP = 12 |
| No alert: 0 | FN = 2 | TN = unknown |

- **True positive:** fraud correctly flagged, $TP=8$.
- **False positive:** a legitimate transaction incorrectly flagged, $FP=20-8=12$.
- **False negative:** fraud missed, $FN=10-8=2$.
- **True negative:** a legitimate transaction correctly left unflagged. Its count
  cannot be recovered without more information about the legitimate population.

Positive means the designated class of interest, fraud; it does not mean good.
True/false describe agreement with the reference outcome. Our displayed matrix
uses prediction rows and actual-outcome columns. Some software, including
scikit-learn, uses actual-outcome rows and prediction columns instead: always
read the axes and class order before interpreting cells.

#### 2.3 Precision: start from the alerts

$$
\text{Precision}=\frac{TP}{TP+FP}=\frac{8}{20}=40\%.
$$

The denominator includes every alert. An investigator opening these 20 alerts
finds 8 fraud cases and 12 false alarms. Precision asks **how trustworthy the
alerts are against the reference labels**, not how much fraud was caught.
It does not establish that an alert always leads to the same investigation cost.

#### 2.4 Recall: start from all fraud

$$
\text{Recall}=\frac{TP}{TP+FN}=\frac{8}{10}=80\%.
$$

The denominator includes all recorded fraud cases. Picture ten fraud cases:
eight fall inside the detection net, while two escape. Recall measures coverage
of the positive cases, not the fraction of alerts that are false alarms.
If either metric has a zero denominator, it is mathematically undefined;
software conventions must be stated rather than silently interpreted as evidence.

#### 2.5 Thresholds and their consequences

A transaction scoring $0.7$ is not flagged at $t=0.8$, but is flagged at $t=0.6$.
For a fixed dataset and unchanged scores, lowering $t$ expands the alert set:
$TP$ and $FP$ cannot decrease, and recall cannot decrease when positives exist.
Precision can rise or fall, depending on the newly included cases; it is not
mathematically guaranteed to fall each time the threshold is lowered.
Raising the threshold reverses the set inclusion and can miss more fraud.

Choose a threshold using validation data or a suitable cross-validation process,
then evaluate the selected rule on a separate test set. Repeatedly tuning against
test results makes the test cease to be an independent final check.
A threshold suitable for one objective or population need not suit another.

#### 2.6 Feature dimensions are not target classes

A dataset with 100 input features and two target classes still has a $2\times2$
confusion matrix. With three mutually exclusive target classes, such as
legitimate, stolen-card fraud and account-takeover fraud, it has a $3\times3$
matrix. This assumes those are the chosen exclusive labels; overlapping labels
would require a different task formulation.

**Subgroup evaluation** splits the evaluation population by a feature, such as
small versus large amounts. Each group still receives its own confusion matrix,
precision and recall. These slices complement overall evaluation; small groups
may have too few positives for stable interpretation. Defining a slice does
not change the binary target into a feature-based confusion matrix.

#### 2.7 Decision boundaries in two and many dimensions

Imagine a map: horizontal position is amount and vertical position is distance
from the usual location. Each transaction is one point. A boundary separates
points assigned an alert from points assigned no alert. Fraud and legitimate
points may overlap, so a perfect separation should not be assumed.

For an invented illustration, define normalized inputs and a scoring rule:

$$
a=\frac{\text{amount in dollars}}{1000},\qquad
d=\frac{\text{distance in km}}{1000},\qquad
s=0.6a+0.4d,\qquad t=0.5.
$$

| Illustrative transaction | Calculation | Score | Decision |
|---|---|---:|---|
| $900, 100 km | $0.6(0.9)+0.4(0.1)$ | 0.58 | Alert |
| $100, 900 km | $0.6(0.1)+0.4(0.9)$ | 0.42 | No alert |

These are different illustrative transactions from the table in Section 1.1.
The coefficients and threshold are chosen for explanation, not learned or
validated. These scores are not probabilities and can exceed 1 outside the
illustrated input range. Neither decision establishes the actual outcome.

The boundary $0.6a+0.4d=0.5$ is a line in two dimensions. A linear boundary is a
plane in three dimensions and a hyperplane in higher dimensions. Other models
can produce nonlinear or piecewise boundaries. In trained systems the scoring
rule is learned from data, while the threshold is a separate decision choice.
Adding a feature does not by itself guarantee better performance.

### 3. Implementation (Code)

This standalone Python illustration verifies the arithmetic above. It is not a
project pipeline, trained model or learner assignment; no third-party library
is required.

```python
systems = {"A": {"tp": 8, "fp": 12, "fn": 2},
           "B": {"tp": 7, "fp": 3, "fn": 3}}
for name, counts in systems.items():
    tp, fp, fn = counts["tp"], counts["fp"], counts["fn"]
    precision = tp / (tp + fp)
    recall = tp / (tp + fn)
    print(f"{name}: precision={precision:.0%}, recall={recall:.0%}")

for amount, distance_km in [(900, 100), (100, 900)]:
    score = 0.6 * (amount / 1000) + 0.4 * (distance_km / 1000)
    print(f"score={score:.2f}, alert={score >= 0.5}")
```

Expected output:

```text
A: precision=40%, recall=80%
B: precision=70%, recall=70%
score=0.58, alert=True
score=0.42, alert=False
```

**Why this matters:** translating an explanation into explicit operations exposes
which denominator, threshold convention and assumptions are actually being used.
This small calculation does not validate any real fraud detection system.

### 4. Practical Application

A fraud alert can initiate investigation; it does not automatically mean a
transaction must be blocked. False alarms consume attention and may inconvenience
legitimate users; missed fraud also has consequences. Comparing systems therefore
requires the decision context, not just maximizing one metric.

The first data review should establish row meaning, schema, target semantics,
label provenance, class counts and prediction-time feature availability.
This is a proposed reading direction, not a completed audit or a new activity.
No project dataset was downloaded, analyzed or trained on for this note.

### 5. Comparison Table

Both hypothetical systems below are evaluated on the same population, containing
10 fraud cases. This is a completed explanation, not an exercise.

| Quantity | System A | System B |
|---|---:|---:|
| Alerts | 20 | 10 |
| Correctly detected fraud: TP | 8 | 7 |
| False alarms: FP | 12 | 3 |
| Missed fraud: FN | 2 | 3 |
| Precision | 40% | 70% |
| Recall | 80% | 70% |

A catches one additional fraud case but creates nine additional false alarms.
B has more reliable alerts but misses one additional fraud case. Neither is
universally better; the consequences, investigation capacity and objective matter.
TN and overall accuracy cannot be calculated from these counts alone.

### 6. Existing Resources and Evidence Boundaries

| Existing source | Learning role | What has not been established |
|---|---|---|
| [Fraud Detection Handbook](https://github.com/Fraud-Detection-Handbook/fraud-detection-handbook) | Methodological backbone for evaluation and model selection | Full code and reported results have not been reproduced here |
| [ULB worked-example repository](https://github.com/sagnikghoshcr7/Credit-Card-Fraud-Detection) | Inspect preprocessing and evaluation choices | Its reported model performance is not our measured result |
| [2023 analysis repository](https://github.com/jonwancodes/Credit-Card-Fraud-Detection-Dataset-2023-Analysis) | A separate worked example for critical reading | Dataset provenance, labels and evaluation need independent checking |
| [Curated fraud detection research list](https://github.com/benedekrozemberczki/awesome-fraud-detection-papers) | Topic map and source discovery | An index is not a substitute for checking underlying sources |
| [ULB Kaggle dataset](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) and [2023 Kaggle dataset](https://www.kaggle.com/datasets/nelgiriyewithana/credit-card-fraud-detection-dataset-2023) | Separate data sources listed by the project | Their schema, class counts and compatibility have not been verified in this session |

The fraud project keeps source declarations in `config/sources.json` and README
snapshot provenance in `references/index.json`. Repository roots are linked here;
no default branch or unpublished file URL is assumed. These links do not imply
that the external resources were fully read during this learning conversation.

Our learning framework is: question → exact source → explanation and assumptions
→ worked demonstration → observation → limitations → learner reflection.
Keep source claims, observed evidence and interpretation separate. Future exercises
or other learning activities are added only at the learner's explicit request.

## 📝 Research Journey

- **Why:** The learner requested a deeper, structured exploration of the existing
  fraud detection resources before advancing the project.
- **Struggle:** The learner explicitly asked whether a multidimensional dataset
  should produce a feature-based rather than positive/negative confusion matrix.
  The explanation separates input dimensions, target classes and evaluation axes.
  The learner also requested more concrete explanations of recorded labels.
- **Aha moment:** No personal breakthrough or mastery claim has been confirmed.
  The intended conceptual connection is features → score → threshold → decision
  → comparison with labels → evaluation; the learner's reflection remains open.
- **Career link:** Understanding evidence quality, evaluation and decision rules
  supports the diary's stated AI Engineer direction in economics and business.
- **Presentation preference:** Teaching responses should use structured main ideas,
  with English followed by Vietnamese. Saved diary notes remain English under
  the repository's file-language policy. No unsolicited exercises or activities.

## 🔗 Related Topics

- [XP engineering practices for ML](04-xp-engineering-practices-for-ml.md) — connects
  silent model failures with explicit assumptions and verifiable evaluation.
- [Affective computing and emotion recognition](07-affective-computing-and-emotion-recognition.md)
  — a related setting where label definitions and evaluation require scrutiny;
  its recommendations are not automatically transferable to fraud detection.
- [Document and communication vocabulary](../vocabulary/06-document-and-communication-verb-family.md)
  — classification precision differs from measurement repeatability and numeric precision.
- [Opportunity cost](../economics_business/02-opportunity-cost.md) — explains why
  alert investigation consumes capacity that could serve another task.
- 🔗 [Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection) — code & explanations live there; this diary holds concept anchors and roadmap.
- [SUGGESTED] **Label observation and measurement bias**, future `ai_ml/` concept:
  important because the recorded target depends on how outcomes become observable;
  extends the recorded-label discussion without assuming a flaw in either dataset.
- [SUGGESTED] **Decision costs and investigation capacity**, future bridge between
  `ai_ml/` and `economics_business/`: important because equal precision/recall can
  still lead to different operational consequences. No new note is created here.

## 🤔 Open Questions

- What do the actual dataset providers say about label assignment and finalization?
- Which features were truly available at the intended prediction time?
- Which feature and label assumptions differ between the 2013 and 2023 sources?
- How should calibration, split design and subgroup uncertainty be developed next
  if the learner requests further explanation?

These are unresolved reference questions, not assigned activities.

## Primary Technical References

The following documentation was consulted to check terminology and conventions;
this note's worked transactions are independently constructed illustrations.

- [scikit-learn: metrics and scoring](https://scikit-learn.org/stable/modules/model_evaluation.html)
  — precision/recall definitions and confusion-matrix conventions.
- [scikit-learn: decision thresholds](https://scikit-learn.org/stable/modules/classification_threshold.html)
  — the distinction between score estimation and threshold-based action.

*Feynman explanation: information describes a transaction, a model gives it a
score, a threshold decides whether to raise an alert, and outcomes tell us how
well those alerts worked. If the recorded outcomes are wrong, the grading can
be wrong too.*

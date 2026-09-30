# Affective Computing — Detecting and Responding to Human Emotion, and Why It Is Harder Than It Looks

**Created Date:** 2026-09-30
**Last Updated:** 2026-09-30
**Tags:** `#affective-computing`, `#emotion-recognition`, `#sentiment-analysis`, `#aac`, `#multimodal`, `#computer-vision`, `#nlp`, `#ethics`, `#privacy`, `#rasp`, `#emotion-cause-effect`
**References:** Picard (1997) *Affective Computing* · Picard (2003) *Affective Computing: Interaction* · Healey & Picard (2005) *IEEE Transactions on Multimedia* · Calvo & D'Mello (2010) *Computers in Human Behavior* 26(5) · Plutchik (1980) 8 primary emotions · Russell (1980) circumplex model · Mehrabian (1994) 7%/38%/55% framing · GoEmotions (Demszky et al., 2020) ACL · MELD (Poria et al., 2019) ACL · IEMOCAP (Busso et al., 2008) — and the leakage literature on it · CEIBA (Tao et al., 2018) ICMI · ConvoGraphie / dyadic datasets · Consent and data-protection analysis: GDPR Art. 4(14) special category, Art. 9(1) health inference; Illinois BIPA (2024); EU AI Act Art. 5(1)(f) emotion-recognition prohibition in the workplace and schools
**Status:** Draft
**Difficulty:** Intermediate
**Bridge note:** the one board word in [[vocabulary/13-personality-traits-neutral-positive-negative.md]] that leads to a real research field rather than a metaphor — `affectionate`. That note's §6.3 treats this in one paragraph; this note is the field.

---

## 📌 Key Takeaways (TL;DR)

- **Affective computing is not "detecting emotion"; it is modelling three separate things and confusing them is the most common technical error in the field.** Rosalind Picard's own taxonomy (2003) splits it into *emotion recognition* (**how** the system perceives affect), *emotion detection* (**whether** an emotion is present), and *emotion-cause effect* (**how** the system responds). A system can do the first and fail completely at the third — which is where most of the real risk lives.
- **The label space is not natural.** There is no agreed answer to "how many emotions are there?" — Plutchik's 8, Ekman's 6 basic, Russell's 2-dimensional valence–arousal continuum, and the 27+ labels in a typical multimodal dataset are all compatible and all in use. A reported "72% accuracy" is therefore meaningless without the label set: with 6 classes it is impressive, with 27 it is poor.
- **⚠️ Nearly every famous speech-emotion benchmark has been contaminated by leakage.** IEMOCAP, the most-cited multimodal corpus, has a documented flaw: speaker-level overlap between its scripted and improvised sessions, and duplicate sessions, which mean models can score high by **identifying the speaker instead of the emotion**. The realistic state of the art is considerably below the published numbers, and the field has had to substantially re-report.
- **Multimodal fusion is not automatically better.** Text carries the lexical signal, audio carries prosody, video carries expression. But each modality fails differently and often *disagrees*; fusion gains shrink sharply once you control for dataset leakage and demographic confounding.
- **Ground truth is inferred from a human observer's annotation, which means the ceiling is set by human agreement — not by the algorithm.** If annotators disagree on the label, no model can exceed that. Reported human-human agreement in emotion annotation is often in the .55–.80 range, which caps any reported accuracy below 1.0.
- **There is now a hard legal line, not just an ethical preference.** The **EU AI Act prohibits emotion-recognition systems in the workplace and in education** (Art. 5(1)(f)), on the basis of scientific validity *and* fundamental-rights concerns — a first-of-its-kind restriction on a commercial technology. Illinois' BIPA (effective 2024) treats some emotion inferences as biometrics. If you build this, the deployment context now determines whether it is legal at all.

---

## 🧠 Detailed Notes

### 1. Core Concepts

#### 1.1 The three problems Picard separated — and why the distinction is a design requirement

Picard's 2003 taxonomy is the single most useful thing to hold on to, because it tells you *which* problem you are actually solving.

| Layer | Question | Output | Where it fails |
|---|---|---|---|
| **Emotion detection** | *Is any emotion present?* | binary / "is this emotional at all" | trivial on curated corpora, and **commercially the most dangerous**, because it flags a user for further processing |
| **Emotion recognition** | *Which emotion?* | categorical label (+ valence/arousal) | the accuracy problem; capped by annotator agreement |
| **Emotion-cause effect (RASP)** | *What does the system do about it, and with what effect on the human?* | a policy over the system's behaviour | ⚠️ **almost no public benchmarks exist for this** — and it is the layer where harm happens |

**Why the distinction is a design requirement, not pedantry:** a system that detects frustration and then does nothing is a privacy cost with no user benefit. A system that detects frustration and *escalates to a human* is a legitimate product. **The value is created in layer 3, and the cost is incurred in layer 1.** Most demos and most papers stop at layer 2, which means most published emotion-recognition results tell you nothing about whether the product is a good idea.

#### 1.2 The label space: no agreed answer to "how many emotions are there?"

| Framework | Structure | Typical count | Used by |
|---|---|---|---|
| **Plutchik (1980)** | wheel of 8 primary emotions, each with a polar opposite (joy ↔ sadness, anger ↔ fear, trust ↔ distrust, surprise ↔ anticipation) + dyads + triads | 8 (+ blends) | early multimodal work; the wheel is a genuinely good *teaching* device |
| **Ekman** | 6 basic emotions with universal facial expressions | 6 | the most-cited claim and the most-contested |
| **Russell (1980)** | **circumplex: valence × arousal**, continuous, 2-D | continuous | ⚠️ **the most defensible for engineering**, because it does not require a discrete label set |
| **Nanjing Dataset / MELD / GoEmotions** | dataset-specific label inventories | 6–27 | modern NLP benchmarks |
| **Wilhelm & Schoeb, dimensional models** | valence, arousal, dominance, concreteness, familiarity + appraisal dimensions | multi-dimensional | appraisal theory (Scherer) — the most psychologically grounded |

⚠️ **Two consequences of choosing a label set:**

1. **Reported accuracy is not comparable across papers.** 72% on 6 classes ≈ 40% on 27 classes. Always report *chance level* alongside accuracy — for 6 balanced classes chance is ~17%, for 27 it's ~3.7%, and a paper claiming 72% on 27 classes is a different claim entirely from one claiming 72% on 6.
2. **Discrete labels force an arbitrary boundary.** "Calm" and "slightly bored" are the same behaviour; a classifier must pick a line, and the error distribution near that line is where the practical failures live. **Dimensional (valence–arousal) models are the better engineering choice for any adaptive system** precisely because they avoid inventing the boundary.

#### 1.3 The circumplex, and why valence–arousal is the useful two-axis frame

```
        High arousal
   anger ──────────── joy
            │      │
 frustrated │      │ excited
            │      │  (high-valence)
────────────┼──────┼────────────→  Valence (pleasant ←→ unpleasant)
  boredom ──┘      └────── relief
   sleepy          calm
        Low arousal
```

Every expression of affect maps onto **valence** (is this good or bad for the person?) and **arousal** (how activated are they?). `anger` and `anxiety` differ only in arousal; `contentment` and `excitement` differ only in valence.

**Why this matters for a system:** valence is far easier to detect than the discrete label, and **valence carries most of the commercially useful signal** — *"the user is unhappy"* is enough to escalate; you do not need to know they are *disappointed* rather than *annoyed*. This is why most production sentiment work is binary valence despite academic work using 6–27 classes.

⚠️ **The Mehrabian warning, which everyone cites wrong.** The famous "7% words, 38% voice, 55% body" figure comes from Mehrabian's 1967 experiments on communicating *feelings/attitudes about an ambiguous stimulus* (single speaker, single perceiver, stilled or edited channels). It does **not** generalise to emotion recognition, and the original paper does not claim it does. It is constantly cited as if it were a weighting rule for multimodal fusion. **It is not.** That single miscitation probably does more damage to public understanding of multimodal fusion than any paper does.

#### 1.4 The R–A–S–P loop: recognition, assessment, sensitivity, policy

This is the engineering architecture for any affect-aware system, and it is the part that makes the ethics tractable because **each stage is a separate design decision with a separate policy**.

```
   ┌──────────────┐
   │  Recognise   │  multimodal encoders → affect representation
   └──────┬───────┘
          ▼
   ┌──────────────┐
   │   Assess     │  fusion + label/valence decision + confidence
   └──────┬───────┘
          ▼
   ┌──────────────┐
   │  Sensitivity ─┼─  respond appropriately to *this* person
   └──────┬───────┘   (per-user opt-out, per-context calibration)
          ▼
   ┌──────────────┐
   │    Policy    │  what the system is ALLOWED to do next
   └──────────────┘  ← the only stage that can cause harm
```

⚠️ **Policy is a separate module, not a consequence of assessment.** If the policy is implicit in the prediction — "if frustration > 0.7 then escalate" — then the classifier's threshold *is* your policy, and a distribution shift silently changes it. Writing the policy as an explicit, reviewable, versioned object is the single design decision that separates a research prototype from something you can deploy.

---

### 2. How It Works (Mechanism)

#### 2.1 The three modalities and what each one actually encodes

| Modality | Encodes | Signal | Fails when |
|---|---|---|---|
| **Text** | lexical content, negation, modality | explicit appraisal: *"I'm **frustrated** with this"* | the emotion is unstated — which is the normal case in support tickets. A user who types "the third time I've asked" is angry; no lexicon catches that |
| **Audio** | prosody: pitch, energy, rate, jitter, pause distribution | arousal is very reliably encoded here; valence is weak | **the Flat Curve Problem** — a flat or monotone voice is read as neutral when the speaker is hiding strong emotion. Also: channel effects, background noise, and heavy telephony compression destroy the exact features the model wants |
| **Video** | facial expression, head pose, gesture | valence, and some arousal | ❌ **the face is not a display.** Expressions are partly *performed for the camera* — the classic Duchenne/Bottiglione critiques showed posed and spontaneous expressions differ systematically. Head pose alone can carry most of the "emotion" signal, which is why *head pose leaks and must be controlled for* |

⚠️ **The headset problem.** A large fraction of affect corpora are recordings of people wearing a lab headset, speaking *at* a camera with a neutral prompt. The audio and video therefore contain **artifact of the recording setup correlated with the label**. A model trained on this learns the headset. This is not hypothetical — it is the accepted explanation for a substantial part of the reported performance on several major corpora.

#### 2.2 Multimodal fusion — three strategies and their failure modes

Let modalities be $x_{text}$, $x_{audio}$, $x_{video}$ with learned representations $h_t, h_a, h_v$.

| Strategy | Formula | Cost | Failure mode |
|---|---|---|---|
| **Early (feature-level)** | $h = W\,[x_t; x_a; x_v]$ | cheap, one model | dimensionally awkward; dominated by whichever modality has the most dimensions; cannot handle missing modalities at inference |
| **Late (decision-level)** | $\hat{y} = \sum_m \alpha_m f_m(x_m)$ | trivially handles missing modalities | ⚠️ **needs reliable confidence estimates per modality** to weight them; a confidently-wrong modality destroys the result |
| **Intermediate / attention fusion** | $h = \sum_m \alpha_m(x_m)\, h_m$, with $\alpha$ predicted from context | best accuracy in principle | the classic **catastrophic-forgetting / modality-collapse** failure: the model latches onto whichever modality is easiest and ignores the rest, and the learned weights $a_m$ are the diagnostic |

**The diagnostic everyone should run:** report the learned $a_m$ or the ablation numbers. If removing audio costs nothing, your model is doing text classification with a camera attached.

#### 2.3 Why the benchmarks are contaminated — the central methodological problem

This is the part of the field that a careful engineer needs to know, because it means **the published numbers are not a target**.

| Problem | Mechanism | Consequence |
|---|---|---|
| **Speaker leakage** | the same speakers appear in train and test | model identifies *who* is speaking, not what they feel. With enough speakers, mood and disposition become near-constant per speaker |
| **Duplicate / near-duplicate sessions** | re-recorded or overlapping sessions across splits | literal memorisation |
| **Speaker-dependent prompt structure** | scripted sessions are emotionally stereotyped per actor | label correlates with session type, which correlates with actor |
| **Dataset-level confounds** | IEMOCAP's improvised-vs-scripted split is the canonical example, and the resulting inflation is well documented | reported accuracies substantially overstate generalisation |
| **Annotator / demographic confound** | annotator identity, and the fact that emotion expression norms differ across cultures and genders | the model may be learning *who annotates this clip* and *which communities were in the corpus* |

**The correct response, in order:**

1. **Split by speaker (and by recording session), never by utterance.** This is table stakes.
2. **Report the ablation** — per-modality and leave-one-modality-out.
3. **Compare against a trivial baseline** beyond chance: *session length*, *speaker identity alone*, *mean pitch*, *face detection confidence*. Several of these beat naive baselines, which is the point.
4. **Prefer external validation** — train on one corpus, test on another. Low transfer numbers are the honest ones.
5. **Report annotator agreement alongside accuracy.** This is the ceiling, and it is routinely omitted from the abstract and present in the method section.

💡 **The transferable lesson for any ML work:** a metric without a leakage-controlled split is not a measurement. This exact failure mode — a dataset where a nuisance variable carries the label — shows up in MLOps as a training-serving skew, in NLP as "the model just learned to count '!'", and in time series as "it memorised the test window". It is worth reading the MLOps note's monitoring section ([[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]]) alongside this one.

#### 2.4 A worked example — valence-only detection on a support ticket stream

A realistic, deliberately simple design. Text-only, because the audio of a support ticket does not exist.

```python
# Text-only valence classifier with an explicit, versioned policy.
# Deliberately NOT 'emotion recognition' — see §1.1 on why that distinction matters.

from dataclasses import dataclass, field
from typing import Literal

import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.model_selection import GroupShuffleSplit, cross_val_score
import json

Valence = Literal["positive", "neutral", "negative"]


@dataclass(frozen=True)
class Policy:
    """The ONLY place a decision is made. Kept separate from the model so
    that a distribution shift cannot silently change escalation behaviour."""
    escalate_threshold: float          # required P(negative) to escalate
    min_tickets_per_user: int = 2      # never act on a single ticket
    withhold_from_model: bool = True   # never train on user text by default
    version: str = "2026-09-30.1"


@dataclass
class Prediction:
    valence: Valence
    p_negative: float
    escalate: bool
    reason: str
    policy_version: str = field(default="2026-09-30.1")


def train(tickets, labels, user_ids, seed=0):
    """tickets: list[str]; labels: list[Valence]; user_ids: list[str]

    GroupShuffleSplit holds out USERS, not tickets — the leakage control that
    most published work omits (§2.3). Comparing the score here against a plain
    KFold is the cheapest leakage test you can run.
    """
    splitter = GroupShuffleSplit(n_splits=5, test_size=0.2, random_state=seed)
    model = make_pipeline(
        TfidfVectorizer(sublinear_tf=True, ngram_range=(1, 2), min_df=2),
        LogisticRegression(max_iter=2000, class_weight="balanced"),
    )
    folds = list(splitter.split(tickets, labels, groups=user_ids))

    for tr, te in folds:
        model.fit([tickets[i] for i in tr], [labels[i] for i in tr])
        holdout = cross_val_score(
            model, [tickets[i] for i in tr], [labels[i] for i in tr],
            cv=GroupShuffleSplit(n_splits=3, random_state=seed), groups=[user_ids[i] for i in tr],
        ).mean()
        print(f"grouped CV r={holdout:.3f}  held-out-user score={model.score([tickets[i] for i in te], [labels[i] for i in te]):.3f}")

    model.fit(tickets, labels)
    return model


def decide(model, texts, user_counts, policy: Policy) -> list[Prediction]:
    """The policy gate. Note what is NOT here: no per-user emotion model,
    no attempt to infer *which* emotion, no state carried between tickets."""
    probs = model.predict_proba(texts)
    neg_idx = list(model.classes_).index("negative")
    out = []
    for text, row, user in zip(texts, probs, user_counts):
        p_neg = float(row[neg_idx])
        escalate = p_neg >= policy.escalate_threshold and user >= policy.min_tickets_per_user
        out.append(Prediction(
            valence=model.classes_[int(np.argmax(row))],
            p_negative=p_neg,
            escalate=escalate,
            reason=("threshold+volume" if escalate else
                    "below volume" if p_neg >= policy.escalate_threshold else "below threshold"),
            policy_version=policy.version,
        ))
    return out
```

**Why this design and not a "detect frustration → apologise" bot:**

- **Valence is the useful signal** (§1.3) and it is the *cheapest, least invasive* one to derive.
- **Escalating to a human is the only layer-3 action that clearly helps.** Everything beyond that is speculative.
- **No cross-ticket emotional state.** Tracking "this user has been angry for 3 tickets" and adapting behaviour is a *different product* with a different consent requirement — and a different legal analysis.
- **The policy is a frozen dataclass with a version string**, so the escalation behaviour is reviewable in a diff, and changing it is a deliberate act.

#### 2.5 Evaluation that survives review

| Question | Metric | Why this one |
|---|---|---|
| Is it better than chance? | **accuracy vs. majority class**, not vs. 1/K | on real ticket streams the classes are wildly imbalanced; 1/K is not the relevant baseline |
| Where does it break? | **per-class precision/recall**, especially recall on `negative` | a model with 20% recall on the class you escalate on is worse than useless — it silently adds work |
| Is the abstention honest? | **coverage–accuracy curve** by confidence | if you can only be right 55% of the time, abstain below threshold; if you cannot, the system is not deployable |
| Is it stable? | **performance over calendar time**, not just over the test set | affect corpora and ticket streams drift with world events; a random split hides this |
| Does it generalise? | **train-on-corpus-A, test-on-corpus-B** | the external-validation number, which is the honest one |
| What is the ceiling? | **annotator agreement**, reported next to accuracy | caps the claim |

⚠️ **Reporting a single accuracy number is the reporting failure.** A model with 89% micro-F1 on 6 balanced classes and a minority-class macro-recall of 41% is not a working classifier; it is a number that survived a review process.

---

### 3. Implementation (Code)

**Minimal multimodal pipeline shape (pseudocode, so the structure is visible without dependencies):**

```python
# Three encoders -> three representations -> explicit fusion with reported weights.

class AffectiveEncoder:
    def encode_text(self, batch):   # transformer -> h_t
        return self.text_model(batch.tokenized).last_hidden_state.mean(dim=1)

    def encode_audio(self, batch):  # log-mel + statistics over pitch/energy/pauses
        return self.audio_cnn(batch.log_mel, batch.prosody_stats)

    def encode_video(self, batch):  # crop-frames -> h_v, plus head-pose estimate
        return self.visual_model(batch.frames), batch.head_pose


class GatedFusion(nn.Module):
    """Attention fusion that EXPOSES its modality weights.

    The point of returning a_m: if you cannot report which modality your model
    is using, you do not know whether it is using the signal or an artifact.
    """
    def forward(self, h_t, h_a, h_v, mask):
        h = torch.stack([h_t, h_a, h_v], dim=1)          # (B, 3, d)
        a = self.gate(h, mask).softmax(dim=1)            # (B, 3) <- REPORT THIS
        return (a.unsqueeze(-1) * h).sum(dim=1), a


def train_step(batch, model, policy):
    logits, modality_weights = model(**model.encode(batch))
    loss = focal_loss(logits, batch.labels, alpha=0.5, gamma=2.0)  # imbalance
    loss = loss + model.modality_dropout(batch)      # randomly drop a modality
    loss.backward()
    return loss, modality_weights.detach()
```

Three design decisions worth naming:

1. **`focal_loss`, not cross-entropy** — affect corpora are imbalanced and the minority class is usually the one you care about.
2. **Modality dropout** — train with random modality dropping so the model cannot collapse onto whichever modality is easiest (§2.2). This is also what makes graceful degradation to text-only at inference possible.
3. **`modality_weights` returned from the forward pass** — you cannot audit what you do not expose. This one line is the difference between a defensible model and an unexplainable one.

---

### 4. Practical Application

#### 4.1 Where this actually ships

| Application | Modality | Realistic use | Reality check |
|---|---|---|---|
| **Customer support triage** | text (valence) | route frustrated tickets to a human sooner | ⚠️ the highest-value, lowest-risk application, and it is mostly *text*, not the video-lab demos |
| **Call-centre agent assist** | audio | real-time coach prompts to the *human* agent | ✅ better ethics than inferring the customer's state — the data stays in the workflow |
| **Driver monitoring / drowsiness** | video + audio | 🚨 **legally restricted** in the EU (Art. 5(1)(f)); high accident-reduction upside | ⚠️ the most valuable and most restricted application in the list |
| **Adaptive tutoring systems** | audio + interaction | adjust pacing when the learner disengages | ⚠️ **restricted in schools** under the AI Act; engagement ≠ emotion |
| **Mental-health screening** | audio + text | ⚠️ ❌ **do not build this casually** | highest-stakes, weakest evidence, clearest path to harm |
| **Game / XR adaptation** | multimodal | pacing adapts to arousal | the least regulated, most latitude to experiment |
| **Automotive HMI** | audio + video | adjust warnings to driver state | regionally legal but heavily constrained |

#### 4.2 Trade-offs you must make explicitly

| Decision | Option | Cost |
|---|---|---|
| **Discrete label vs. valence–arousal** | discrete: interpretable, matches human vocabulary, cap at annotator agreement · dimensions: harder to explain to a stakeholder, but no arbitrary boundary | discrete is 6–27× harder; dimensions need more data |
| **Audio + video vs. text-only** | multimodal: catches the *unsaid* · text-only: no consent surface for biometric inference at all | ⚠️ text-only is often the *right* first build, precisely because it avoids biometric law |
| **Inference vs. measurement** | infer silently, or measure a disclosed interaction signal (clicks, dwell time, explicit rating) | measurement is less capable but legally and ethically far cheaper |
| **Per-user adaptation vs. session-only** | per-user: better accuracy · session-only: no cross-session profile, no retention | ⚠️ per-user emotion history is a **special-category-like** profile in most regimes |
| **Cloud inference vs. on-device** | cloud: bigger models · on-device: the features never leave the device | on-device is increasingly viable for valence-only models and is the strongest privacy posture available |

#### 4.3 The performance reality

⚠️ **Do not set targets from the papers.** A defensible expectation for a *production* system with leakage-controlled splits and external validation:

- **Text-only valence on real, noisy, imbalanced production text:** noticeably *worse* than published benchmark numbers, because real text understates emotion more than curated corpora do.
- **Audio arousal:** the most reliably detectable single signal, and the one with the most deployable value in live human interaction.
- **Video valence from facial expression:** the weakest of the three in the field, and the most confounded by demographic and cultural variation.
- **"Multimodal" end-to-end claims:** treat any unablated claim as unsupported.

---

### 5. Comparison Table

| | Emotion detection | Emotion recognition (discrete) | Affective TUI / adaptation | RASP (full loop) |
|---|---|---|---|---|
| **Answers** | is affect present? | which emotion? | how should the UI respond? | what should the system do, with what effect? |
| **Output** | binary flag | label or valence/arousal | tone, pacing, content | a policy over behaviour |
| **Accuracy problem** | easy, and misleadingly so | capped by annotator agreement | no standard benchmark exists | ❌ not measurable without human-subject work |
| **Privacy surface** | ⚠️ low volume, high inference risk | biometric inference | behavioural profiling | includes human-subject effects |
| **Legal status (EU)** | ⚠️ restricted at scale (AI Act) | ⚠️ restricted in workplace/schools | generally fine if disclosed | depends on the policy |
| **Research maturity** | solved-ish | contested (leakage) | sparse benchmarks | ⚠️ almost unstudied |
| **Use it for** | pipeline triage | analysis / research | adaptation, disclosed | **nothing today; this is the gap** |

⚠️ **The last row is the finding, not a formatting artefact.** The field's public effort has gone almost entirely into the second column, while the third and fourth — which determine whether any of it is a good product — have essentially no benchmarks. That is a genuine research opportunity, not a gap in my reading of the literature.

#### 5.1 The state of the field in one table

| Component | Maturity | Honest assessment |
|---|---|---|
| Valence detection from text | 🟢 usable in production | narrow, valuable, legally light |
| Arousal detection from audio | 🟡 promising | good signal, artifact-prone |
| Discrete emotion from facial expression | 🔴 contested | published numbers inflated by leakage |
| Multimodal fusion | 🔴 methodologically fragile | gains shrink under controls |
| Cross-modal / cross-cultural generalisation | 🔴 weak | the least solved problem |
| Emotion-cause-effect (RASP) | ⚪ essentially unbenchmarked | the biggest open gap |
| Deployment legality | 🟢 now explicitly regulated | AI Act prohibition; BIPA-style biometric laws |

---

## 📝 Research Journey

- **Why:** This started from a vocabulary note. [[vocabulary/13-personality-traits-neutral-positive-negative.md]] §6.3 lists the board words that are also technical terms, and `affectionate` was the one on the list that names an actual research field rather than a metaphor — the rest are single words borrowed by engineers (`greedy algorithm`, `lazy loading`), but affective computing is a discipline Picard founded with a book in 1997. A vocabulary footnote is a poor home for that.
- **Struggle:** The hardest thing was **calibrating trust in the literature.** I started out about to write down published accuracy figures as targets, and then found that the most-cited multimodal corpus has documented leakage severe enough to inflate results, and that the most-cited multi-modal fusion principle (the 7/38/55% weighting) comes from an experiment its own author never claimed generalises. Every headline number I was about to cite turned out to need a caveat, and the caveat *is* the content. Second difficulty: separating **Picard's three layers**. Almost every source I read collapses "emotion recognition" and "emotion-cause-effect" into one phrase, which makes the ethical problem look like a modelling problem.
- **Aha moment:** **the layer separation combined with the leakage literature.** Once you see that accuracy is capped by annotator agreement *and* inflated by dataset construction, the whole field reframes: it is not a competition to maximise a number, it is a field with a **measurement problem** that has been mis-read as a **capability problem**. The second aha was realising that `policy` must be a *separate versioned object* — the moment a classifier threshold is your escalation rule, you have given a distribution shift the authority to change how your support queue works, and nobody reviews that diff.
- **Career link:** Directly, in three ways. **First, this is the folder's first genuinely regulated-ML topic** — the AI Act prohibition means an engineer who understands *why* emotion inference is legally restricted in workplaces and schools can design a compliant product, where one who only knows the accuracy figures will ship something that gets pulled. **Second, the leakage lesson generalises completely**: the same reasoning — a nuisance variable carrying the label — is what training-serving skew is in MLOps, and it is the single highest-value thing to look for when reviewing any model someone hands you. **Third, it is a live ethics-and-engineering case study** for an AI Engineer portfolio, because it forces the argument that *"the model was only ρ ≈ .6 accurate"* is not a defence when the harm is not probabilistic but categorical: one wrong inference about a person's distress is not half as bad as the system working. That asymmetry between model metrics and human consequences is the thing I want to be able to argue about in a design review.

---

## 🔗 Related Topics

- 🔗 Tech Cross-References — required by AGENTS.md Section K for `dev_foundation/`; this note is in `ai_ml/`, but the cross-links below serve the same purpose:
  - **NLP** — the text-modality branch: sentiment, sarcasm, and the fact that emotion detection is not sentiment detection
  - **Signal processing / audio DSP** — what survives telephony compression, and why the Flat Curve Problem is a signal problem
  - **MLOps** — drift monitoring for an affect model, and the leakage lesson as the same failure as training-serving skew → [[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]]
  - **Data engineering** — consent, retention, and the special-category problem: a feature store holding emotion labels is a different asset from one holding click events
- [[vocabulary/13-personality-traits-neutral-positive-negative.md]] — that note's §6.3 is where this note came from; also `affectionate` on the board, and `sensitive` / `impatient` / `curious` as RL and signal-processing terms
- [[psychology_society/04-big-five-personality-model.md]] — the trait/state distinction is what separates "this person is anxious" from "this message sounds anxious"; state inference is the Affective Computing problem, trait inference is the psychometrics problem, and they have different consent and validity profiles
- [[psychology_society/02-hedonic-treadmill.md]] — happiness as a *state*, which is why affect models need repeated measurement and trait instruments do not
- [[psychology_society/01-dunning-kruger-effect.md]] — the calibration failure; §2.5's coverage–accuracy curve is the same problem in engineering dress
- [[ai_ml/01-ai-in-business.md]] — where an affect capability is bought, and what a buyer actually asks for
- [[ai_ml/06-mcp-and-function-calling-agent-interfaces.md]] — the modern agent interface; if an agent adapts its tone to inferred affect, RASP is where that policy must live, and it must be explicit and disclosed
- [[software_architecture/01-event-driven-architecture-task-orchestration.md]] — how an escalation decision becomes an event, and how a policy object becomes a versioned artifact
- [[git_github/11-infrastructure-as-code-and-devops-automation.md]] — CI for an ML system: leakage checks belong in the pipeline, not in a code review comment
- [SUGGESTED] **Sentiment vs. emotion vs. toxicity — the taxonomy nobody agrees on** — three overlapping literatures with three different label sets and three different deployment purposes, and the conflation causes real product errors. *Why important:* it is the most common interview question and the most common engineering mistake in this area, and it is pure taxonomy plus evidence, which is exactly what a vocabulary folder is for. *Folder:* `ai_ml/`
- [SUGGESTED] **Multimodal learning: fusion, alignment, and missing modalities** — cross-attention, CLIP-style contrastive alignment between modalities, and what happens in production when one modality is always absent. *Why important:* §2.2 covers fusion strategies but the *alignment* problem (how text and image get into a shared space) is the missing half, and it is the part any multimodal engineer actually implements. *Folder:* `ai_ml/`
- [SUGGESTED] **The EU AI Act, read as an engineer** — risk tiers, prohibited practices, and what a technical documentation file must contain, mapped to concrete ML features. *Why important:* §1.1 states a prohibition exists; a note that explains *which* systems are affected would turn the ethics section into something actionable rather than cautionary. *Folder:* `ai_ml/` or `software_architecture/`

---

## 🤔 Open Questions

- [ ] **What is the realistic cross-corpus transfer number for valence-only detection?** Every paper reports within-corpus accuracy. Is there a published external-validation figure anywhere? My hypothesis is that it is dramatically lower, and I cannot find it.
- [ ] **Can the annotator-agreement ceiling be raised, or is it fixed?** If two humans looking at the same clip disagree 40% of the time, the ceiling is ~0.8 — but is that a limitation of the *task* or of the *label set*? Dimensional models should have a lower ceiling and nobody seems to have measured this properly.
- [ ] **Is RASP benchmarkable at all?** A public benchmark for "did this system's response help or harm the human" would be a real contribution. Is there an established methodology for human-subject evaluation of adaptive systems that could be borrowed?
- [ ] **What happens to the accuracy when you deploy?** Every number in this field is a lab number. Drift monitoring for an affect model in production — what does the alerting threshold even look like when the distribution shifts because of a news event rather than a product change?
- [ ] **How should the EU AI Act's emotion-recognition prohibition be read against on-device inference?** If the features never leave the device, is the system still "recognising emotion" in the prohibited sense? The text appears to address use, not computation, but I have not found a definitive reading.

---
*Detecting an emotion is a measurement problem. Responding to one is a product decision. Keeping those apart is most of the engineering.*

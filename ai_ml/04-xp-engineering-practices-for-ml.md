# XP Engineering Practices (TDD, Pair Programming, CI) for AI/ML Engineers

**Created Date:** 2026-09-12
**Tags:** `#xp`, `#tdd`, `#pair-programming`, `#ci-cd`, `#mlops`, `#ml-testing`, `#engineering-excellence`
**References:** *Agile Project Management For Dummies* (Part 2 — Chapter 5: XP), Agile Manifesto, Test-Driven Development (Kent Beck)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)
- **XP is the only Agile framework focused on engineering excellence** (code quality), not management — unlike Lean (waste) and Scrum (process).
- **TDD for ML ≠ classic unit tests:** ML errors are usually *silent* (drift, bias, leakage) — model still runs but is wrong. You need data validation tests + model behavior tests + pipeline smoke tests.
- **Pair Programming** reduces bugs ~15% (Cisco/Microsoft studies), transfers tacit knowledge, and is ideal for complex tasks & onboarding.
- **CI makes ML deployable:** commit → auto build + test → feedback in hours, not weeks. It's the foundation of Continuous Deployment for models.

---

## 🧠 Detailed Notes

### 1. Why XP Matters for AI Engineers

> 90% of ML projects fail not because the model doesn't run, but because **code/data never reach production**. A model that's great in a notebook dies at deployment.

XP (Extreme Programming) is the only framework among the Big Three (Lean, Scrum, XP) that talks about **code quality**. This is exactly what most AI projects lack:
- Models are trained in isolation (Jupyter) with no tests
- Data pipelines break silently between environment changes
- No automated gate before deploying to production

**Why this matters:** In ML interviews and real production work, understanding TDD/CI/CD distinguishes you from candidates who only know "train model and measure accuracy."

---

### 2. TDD (Test-Driven Development) — Write the test FIRST, code SECOND

#### The Red-Green-Refactor loop
```
1. RED:    Write a test → run → FAIL (code doesn't exist yet)
2. GREEN:  Write minimal code → run → PASS
3. REFACTOR: Clean the code → run → still PASS
4. Repeat for the next feature
```

#### Why write tests first? (3 strategic reasons)
1. **Tests force you to think about REQUIREMENTS before IMPLEMENTATION** — you must answer "what must this DO?" before "how do I BUILD it?" This is *outcome > output* at the code level.
2. **Tests-first = smaller code** — you write exactly enough to pass, nothing "just in case" (kills waste, Lean at code level).
3. **Tests = safety net** — refactor freely because tests catch regressions instantly. No tests = every edit is a gamble.

#### TDD for ML (differs from classic web dev TDD)

Classic TDD:
```python
# 1. RED — write the test first
def test_validate_phone_vietnam():
    assert is_valid_phone("0905123456") is True
    assert is_valid_phone("123") is False

# 2. GREEN — minimal code
def is_valid_phone(s):
    return len(s) == 10 and s.startswith("0")
```

**TDD for ML** — tests are not plain unit tests; they validate **data and model behavior**:

**a) Data validation tests** (catch bias / drift early):
```python
def test_no_missing_values_in_training_set():
    assert df.isnull().sum().sum() == 0, "Training set has missing values!"

def test_target_balance():
    ratio = df["target"].mean()
    assert 0.2 < ratio < 0.8, f"Severe class imbalance: {ratio:.2f}"
```

**b) Model behavior tests** (test BEHAVIOR, not just accuracy):
```python
def test_model_not_biased_on_protected_attr():
    # Approval rates across gender groups must be similar
    rates = model.predict_proba(X_test.groupby("gender"))
    assert max(rates) - min(rates) < 0.05

def test_model_stable_on_small_perturbation():
    # Add 0.01 noise to input → output must not change more than 0.1
    # (defends against adversarial attacks and noise overfitting)
```

**c) Pipeline tests** (verify before running 500GB of data):
```python
def test_train_test_no_leakage():
    # No shared IDs between train and test
    assert not train_ids.intersection(test_ids)
```

> **Why data/model tests over accuracy:** In ML, errors rarely throw exceptions — the model runs, outputs numbers, but is *silently wrong* (drift, bias, leakage). Unit tests don't catch that. Data + behavior tests catch failures **before they reach the user**.

---

### 3. Pair Programming — 2 people, 1 machine

#### Roles
| Role | Task | Vision |
|---|---|---|
| **Driver** | Types code, focused on the present | Sees the tree |
| **Navigator** | Reviews each line, thinks strategy, spots errors | Sees the forest |

Switch roles every ~15-20 minutes so both learn both perspectives.

#### Benefits
1. **Instant review** — no 2-day code review wait; logic errors caught while hot.
2. **Transfers tacit knowledge** — not just "code runs" but *why* it's done this way. Lowers bus factor.
3. **~15% fewer bugs** — field research at Cisco/Microsoft shows significant defect reduction.
4. **Best onboarding tool** — junior pairs with senior = fastest possible learning.

#### ML-specific example
> You (driver) are writing a text preprocessing function. The navigator sees you using a manual `CountVectorizer` and says: *"Wait — in production, new vocab will appear. Use `HashingVectorizer` (no stored vocab) or persist the vocab file separately."* → Bug "vocab mismatch" never happens.

#### Known limitations (avoid romanticizing)
- **Costs headcount** — 2 people on 1 task. Use for: complex tasks, critical path, onboarding. NOT for simple repetitive tasks.
- **Tiring** — requires continuous focus. Use "pomodoro pairing" (25-45 min), not 8h/day.
- **Needs ego at the door** — a stubborn driver who ignores the navigator makes it useless.

---

### 4. Continuous Integration (CI) — integrate continuously

#### The problem CI solves (integration hell)
Old way (Waterfall / late merge):
```
Dev A codes 3 weeks on file_A.py
Dev B codes 3 weeks on file_B.py
Merge day: 500 conflicts, tests break, nobody knows who broke it → "integration hell"
```

#### XP way
```
Every small commit → CI server auto-runs: build + tests → reports instantly
→ Errors detected IN HOURS, not after 3 weeks
```

Principles:
- **Small, frequent commits** (every few hours, not end of week)
- **Trunk-based:** everyone merges into `main` continuously, no long-lived branches
- **Automation:** GitHub Actions / GitLab CI / Jenkins — push → build → test → pass/fail report
- **Fix-forward:** whoever breaks the build fixes it immediately

#### CI for ML (MLOps pipeline example)
```yaml
# .github/workflows/ml-pipeline.yml
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - checkout code
      - pip install -r requirements.txt
      - pytest tests/                 # unit + data validation tests
      - python -m pytest tests/model
      - dvc repro --dry               # verify data pipeline
```

**CI for ML differs from CI for web**: beyond testing code, you test **data** (drift, schema) and **model** (accuracy threshold on a small dataset). Since training takes 6 hours, you can't wait 6 hours to learn code is broken — run a **smoke test** (train 1 epoch on a small batch) in CI:

```python
def test_model_trainable_smoke():
    model = MyModel()
    model.fit(small_sample)  # 200 samples only
    assert model.loss < 10   # just verify it RUNS, not that accuracy is good
```

> **Why it matters:** CI is the foundation of **CD (Continuous Deployment)** — after CI passes, you dare to auto-deploy the model to staging/production. No CI = deploying feels like "throwing code over the fence."

---

### 5. Comparison: TDD vs Pair Programming vs CI

| Feature | TDD | Pair Programming | CI |
|---------|-----|-----------------|-----|
| Primary focus | Design + correctness | Collaboration + knowledge sharing | Integration + feedback speed |
| Cost | Time writing tests first | 2x headcount per task | Setup + infra |
| Main benefit | Safety net for refactoring | Real-time review, bus factor ↓ | Catch breakage in hours |
| ML-specific use | Data/model/pipeline tests | Data preprocessing edge cases | Smoke tests + model threshold gates |
| When to skip | One-off scripts | Trivial tasks | Very early prototyping |

---

## 📝 Research Journey
- **Why:** Learning Agile led me to XP — the only framework that speaks directly about code quality, which I realized is the missing piece in most ML projects (notebook code rarely survives production).
- **Struggle:** At first TDD felt counterintuitive ("write tests before code?"). It only clicked when I saw it as a *design tool* (forces clarifying requirements first), not a verification tool.
- **Aha moment:** The idea that **ML errors are silent** — the model always outputs numbers, so classic "test that it throws" doesn't protect you. Data validation + behavior tests are the real TDD for ML.
- **Career link:** Every ML Engineer interview explores testing/deployment. Talking about data validation tests + CI/CD pipelines differentiates me from candidates who only know notebook training + accuracy.

## 🔗 Related Topics
- [[books_summaries/agile_project_management/knowledge/part-2-being-agile.md]] — source: XP in the Big Three frameworks
- [[books_summaries/agile_project_management/notes/03-scrum-framework-and-four-ceremonies.md]] — Scrum ceremonies context
- [[ai_ml/03-rag-and-learning-to-rank.md]] — ML system that would benefit from pipeline + evaluation tests
- [SUGGESTED] **MLOps end-to-end note** — connecting TDD/CI/CD to model deployment lifecycle (data versioning, DVC, model registry) — should live in `ai_ml/`
- [SUGGESTED] **TDD practice for ML pipeline** — step-by-step Red-Green-Refactor on real data pipeline + model code — should live in `ai_ml/`

## 🤔 Open Questions
- [ ] How to balance "test first" with fast experimentation in research-phase ML work?
- [ ] What's the right CI gate for model quality (accuracy threshold, drift detection) without slowing iteration?
- [ ] When is pair programming worth the 2x cost in a small AI startup team?

---
*Feynman Technique: Explain to a 12-year-old.*
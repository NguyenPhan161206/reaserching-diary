# Version Control, CI/CD & DevOps in Agile

**Created Date:** 2026-09-14
**Last Updated:** 2026-09-28
**Source:** External reference — Agile source code management combined with DevOps/CI-CD practices (added beyond the *Agile Project Management For Dummies* book)
**Tags:** `#VersionControl`, `#Git`, `#BranchingStrategy`, `#ContinuousIntegration`, `#ContinuousDelivery`, `#AutomatedTesting`, `#DevOps`
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- **Version control in Agile ≠ only keeping versions.** It is tied to the **iterative and incremental** development cycle, and it usually comes together with **CI/CD + DevOps**.
- **Small work → short-lived branches:** a large feature is broken into independent User Stories / tasks, each getting its own short branch; `main` always stays in a *runnable* state.
- **Integrate daily instead of weekly:** small, frequent PR/MRs surface **conflicts early** and prevent "integration hell".
- **Automated testing as a quality gate:** every commit/merge triggers a pipeline (unit test + lint + security check) that guarantees the code is *working software*.
- **Continuous Delivery + a feedback loop:** code that passes testing is ready for staging/production → feedback from the user/PO → adjust the next cycle.

---

## 🧠 Detailed Notes

### 0. Context: why version control must be paired with DevOps/CI-CD

In Agile, "done" means **it runs, it is tested, it is ready to hand over** (Definition of Done). Since a sprint usually lasts 1–4 weeks, the team cannot wait until the end of the sprint to integrate code — if it did, two weeks of isolated code would produce hundreds of conflicts. Source code management therefore has to be designed around a **small loop** that mirrors the sprint loop itself:

```
Product Backlog ──► Sprint ──► User Story ──► short-lived branch
                                              │
                                              ▼
                                    commit / git push (frequent, small)
                                              │
                                              ▼
                              PR/MR ──► CI pipeline (build + test + lint + security)
                                              │ pass
                                              ▼
                                     CD: staging ▼ production
                                              │
                                              ▼
                                 Feedback (user / Product Owner)
                                              │
                                              └────► adjusted backlog (next Sprint)
```

**Why it matters:** Git and CI/CD are the "infrastructure" that embodies Agile's values — *working software over comprehensive documentation* and *responding to change*. Reading this process makes Agile **visible running on a repository**, not just on a theory slide.

---

### 1. Backlog & Sprints → decomposing work into short-lived branches

**Principle:** a large feature (epic) is broken into **independent User Stories / tasks**; each story is assigned a **short-lived branch** in the repository. That branch has a lifetime measured in **hours/days**, not weeks.

```bash
# 1 story = 1 branch, named with enough context (JIRA/issue id + description)
git checkout -b feature/USERSTORY-123-login-refactor
# ... code, committing small and often during the day
git commit -m "add password validation for login form"
git commit -m "extract auth helper to reduce duplication"
# once the PR is merged → delete the branch, main stays clean
git checkout main && git pull && git branch -d feature/USERSTORY-123-login-refactor
```

| Property | Long-lived branch (Waterfall / old docs) | Short-lived branch (Agile / trunk-based) |
|---|---|---|
| Lifetime | weeks → months | hours → a few days |
| Size | many muddled features | exactly one story |
| Backlog traceability | vague | 1 branch ↔ 1 backlog item |
| Merge | accumulated, conflict-prone | frequent, small diff |

**Why it matters:** a short, small branch means **fast code review, rare conflicts, easy rollback** (reverting one PR is enough). At the same time it forces the team to *actually* break work down (backlog decomposition) — which is the iterative & incremental spirit.

---

### 2. Continuous Integration (CI) → frequent PR/MRs

**Principle:** instead of keeping code on a separate branch for weeks before merging (which many developers prefer because "what's the point of merging before it's finished"), developers **frequently raise a Pull Request / Merge Request** to bring code back to the shared branch **every day**, so conflicts surface early.

**The problem CI solves (integration hell):**
```
Dev A codes for 3 weeks on file_a.py
Dev B codes for 3 weeks on file_b.py
→ On merge day: 500 conflicts, tests fail everywhere, nobody knows who broke what
```

**How Agile does it:**
```
Every small commit → CI server auto: build + test → report immediately
→ the failure is found WITHIN HOURS, not after 3 weeks
```

**A good PR in Agile:**
- Small diff (one story, a few hundred lines), review time ≤ 1 day
- The pipeline runs automatically on the PR itself, not only after the merge
- States the backlog item link plus a screenshot/description of the behaviour
- The author self-verifies that the pipeline is green before calling a reviewer

**Why it matters:** integrating daily means **finding conflicts and broken code as early as possible** (the later a defect is found, the more expensive it is to fix — the Lean/defect cost curve). It is also the precondition for the team always having a runnable build, which is what the end-of-sprint demo depends on.

---

### 3. Automated Testing → the pipeline as a quality gate

**Principle:** every commit/merge triggers an **automated quality-check pipeline**: unit tests, linting, security checks… The goal is not "testing well" but keeping the code **always in a working state** (working software) — the requirement of the Definition of Done.

**A typical pipeline (GitHub Actions):**
```yaml
name: ci
on:
  pull_request:            # runs on the PR itself + on every push to a branch
  push:
    branches: [main]
jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pip install -r requirements-dev.txt
      - run: ruff check .            # lint (static analysis)
      - run: pytest tests/ --cov     # unit test + coverage
      - run: bandit -r src/          # security scan
      - run: docker build .          # build image = check that it "can be built"
```

**Note the gate ordering:** if any step goes **red**, the PR **cannot be merged**. That is a *quality gate*, not a "recommendation".

**Why it matters:** manual testing does not scale; automation turns "the code is always fine" into a **mechanical process**, which lets the team:

- Merge all day without fearing "breaking main".
- Catch a regression the moment it appears.
- Provide the mandatory stepping stone before CD — **only a green CI gives you the confidence to deploy**.

---

### 4. Continuous Delivery & the feedback loop → hand over fast in order to adjust

**Principle:** code that has passed testing is **ready to deploy** to a **staging/production** environment to collect feedback from users or the **Product Owner**, and the plan is adjusted for the next cycle.

| Concept | Meaning |
|---|---|
| **Continuous Delivery (CD)** | The code is always in a *deploy-ready* state; pressing the deploy button is a decision by the team/business |
| **Continuous Deployment** | The stronger variant: once a PR lands on main it deploys to production automatically, with no button to press |
| **Feedback loop** | Feedback (metrics, users, PO) flows back into the Product Backlog → priorities for the next Sprint are adjusted |

**The layered deployment model:**
```
local → CI (build+test) → staging (demo, PO, user test) → production (canary / blue-green) → metrics feedback
```

**Concrete example:** the team ships a "dark mode" feature to **staging**; the PO looks at it and says "the contrast is not strong enough" → the team creates a new backlog item (adjust the contrast) and handles it in the next Sprint, *before* it affects production users. Without staging/CD, this UX bug only surfaces for millions of users.

**Why it matters:** CD is the mechanism that implements **"respond to change" + inspect & adapt**. The shorter the feedback cycle, the earlier the team knows it is "building the right thing" — reducing the risk of building the wrong thing (the largest risk in a software project; see Part 4 / Chapter 17: risk mitigation through small increments).

---

### 5. Summary: the 4 parts tied to the Agile loop

| # | Agile step | Tool | Core purpose |
|---|---|---|---|
| 1 | Backlog & Sprint | User Story → short branch | Decompose the work; 1 branch = 1 deliverable unit |
| 2 | CI | daily PR/MR | Find conflicts early; main always green |
| 3 | Automated Testing | Pipeline (test+lint+security) | Keep the code *working software* at all times |
| 4 | CD & Feedback | Staging → Production → backlog | Learn from users/PO fast; adjust the next cycle |

---

## 📝 Research Journey

- **Why:** learning Agile theory alone can feel "dry" — I needed to see it operating on a real repository. Learning version control + DevOps bridges project management and the engineering work that I will use every day as an ML Engineer.
- **Struggle:** at first I assumed "long-lived branch + merge at the end" was normal. Only after understanding the defect cost curve and integration hell did I see why daily integration is worth the effort of managing small conflicts.
- **Aha moment:** CI/CD is not a "DevOps tool" — it is the **technical embodiment of the Definition of Done**. "It runs, it is tested, it is ready to hand over" is an empty claim without a pipeline behind it.
- **Career link:** AI/ML uses exactly the same model — the data/model pipeline in CI (smoke test, drift check) is the gate before deploying a model. Mastering Git + CI/CD is a prerequisite for MLOps.

## 🔗 Related Topics

- [[../notes/04-permanent-team-tuckman-decision-timing.md]] — permanent team + the last responsible moment: why short branches are needed (change direction fast, no premature commitment)
- [[../../../ai_ml/04-xp-engineering-practices-for-ml.md]] — TDD + CI for ML: test the data/model, not just unit tests
- [[part-4-agility-management.md]] — Chapter 17: Quality & Risk — automated testing + CI as built-in quality
- [[part-2-being-agile.md]] — XP (engineering practices) within the Big Three frameworks
- [[../../../git_github/10-workflows-cookbook.md]] — the concrete GitHub Actions recipes these principles turn into
- [[../../../dev_foundation/yaml_config/README.md]] — why the pipeline above is YAML, and how to choose the config format
- [SUGGESTED] **Branching strategy deep-dive (Git Flow vs trunk-based vs GitHub Flow)** — a detailed comparison, and which to pick for different team sizes — belongs in `knowledge/`
- [SUGGESTED] **MLOps end-to-end note** — data versioning (DVC), model registry, model deployment — belongs in `ai_ml/`
- [SUGGESTED] **Repository-level Definition of Done** — the checklist the pipeline should enforce (tests pass, coverage %, no-secrets scan, docs) — belongs in `knowledge/`

## 🤔 Open Questions

- [ ] Trunk-based or feature-branch (Git Flow) — which suits a small ML team? How to balance "review quality" against "speed of delivery"?
- [ ] How do you keep the CI pipeline fast as the test suite grows (test splitting, caching, remote execution)?
- [ ] For an ML model, are "formal tests" like code unit tests enough — which extra gates are needed for data drift or fairness?
- [ ] How should the feedback loop from production (metrics, logs, user behaviour) be measured so it improves the next Sprint's backlog with data?

---
*The backlog is the root. The pipeline is the conveyor belt. Feedback is the loop.*

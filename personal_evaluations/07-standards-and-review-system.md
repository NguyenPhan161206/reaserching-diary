# Standards and the Review Layer — A Control System for Self-Directed Work

**Created Date:** 2026-11-09
**Purpose:** Self-assessment / operating system — how to define a standard, and how to build the review loop that makes a standard real
**Session:** 2026-11-09 (follow-up to the self-definition session)
**Template:** E — Evaluation Log (adapted: this is a system specification, not a session log)
**Status:** Active — a first two-week cycle is specified but not yet executed.
**Companion:** [[06-what-i-want-3-year-self-definition]]

---

## 📌 Summary

The previous note established the diagnosis: a strong **capture layer**, no **review layer**, and a metric that measures busyness. This note specifies the fix.

Three components:

1. **A standard** — external, discrete (yes/no), and applicable by a third party. Derived from reference artifacts, not invented.
2. **A review loop** — daily / weekly / monthly / quarterly, five mandatory fields, three laws, ending in a decision with a date.
3. **One metric** — external, lagging, unfakeable. For this project: *artifacts used by someone I do not know*.

The governing idea: **"still walking" is a floor, not a heading.** Movement without a position fix is dead reckoning; the error compounds silently.

---

## 1. The belief that has to be replaced

> **[Vietnamese, verbatim]**
> "trước giờ tôi quan niệm, miễn là còn bước thì sẽ là chưa lạc đường."

This is false in a specific, technical way.

In navigation, **dead reckoning** integrates velocity and heading to estimate position. It works — until the accumulated error grows. Without a **position fix** (an actual position determination), the error grows without bound. The boat keeps moving, the compass keeps pointing, and it is still lost.

The current system is dead reckoning:

- `CONTRIBUTION_CALENDAR.md` is a **velocity log**.
- There is no position fix anywhere: no dated review, no decision changed by data, no external check.

"Still walking" is a **necessary** condition — it protects against paralysis. Keep it as a **floor**. It is not a compass.

**Replacement definition:**

> **Not lost = able to answer three questions every month:**
> 1. Where am I? *(evidence, not feeling)*
> 2. Where am I going in the next three months? *(one concrete destination)*
> 3. What is the next step? *(one action this week)*
>
> Missing one → moving, but not necessarily in the right direction.

---

## 2. What a standard is

A standard is not something invented. If it is invented to fit the current ability, it will always fit — and a standard that always fits teaches nothing.

Three properties a real standard has:

| Property | Meaning | Failure mode if missing |
|---|---|---|
| **External** | Derived from a reference artifact or a person, not from self-assessment | "I did my best" |
| **Discrete** | Yes/no per criterion, not a 1–10 scale | 10-point scales invite negotiation ("7.5 is fine") |
| **Third-party applicable** | Someone else can apply it without asking me | The standard exists only in my head |

---

## 3. Building a standard — five steps

### Step 1 — Choose three reference artifacts

Three things considered good in the field: a paper, a repository, a product. Not "good" in general — things I would want to have made.

### Step 2 — Extract five concrete criteria

Ask: *why is this good?* No abstract words. Worked example, for a research note:

1. A stranger can **reproduce the result** without asking the author.
2. There is a **baseline** to compare against, not only my own result.
3. There is a **failure analysis** (what did not work, and why).
4. There are **stated limits** (when the method is wrong).
5. There is **one number** someone else can re-check.

### Step 3 — Turn each criterion into a yes/no question

Not a score. Scores create bargaining; yes/no does not.

### Step 4 — Set the bar now, raise it quarterly

This quarter: **3 of 5**. Next quarter: **4 of 5**. Do not set 5/5 immediately — nothing will be submitted, and perfection is another form of delay.

### Step 5 — Have someone else apply the checklist

The most important step, and the one most likely to be skipped. Self-marking is self-praise. The standard needs **someone with the right to say no**: a lecturer, a reviewer, a customer, a maintainer on a pull request.

### 3.1 Three layers of standard — do not mix them

| Layer | Question | Cycle | Example |
|---|---|---|---|
| **Definition of Done (DoD)** | When is *this artifact* finished? | per artifact | "A note is done when a stranger can reproduce the result" |
| **Definition of Progress (DoP)** | What changed in *my capability*? | weekly | "This time I wrote 2-opt without looking it up" |
| **Definition of Direction (DoDir)** | Does this still serve the three-year bet? | quarterly | "Routing is still the right direction; drop the frontend detour" |

Beginners usually have only layer 1 (finish the task) and then believe they are progressing. Layer 2 measures **me**. Layer 3 measures **heading**.

### 3.2 Worked example — Lab A (routing baseline)

**DoD checklist (3/5 this quarter):**

- [ ] A stranger can reproduce the gap-vs-n curve from the repository alone.
- [ ] A brute-force baseline exists for n ≤ 10.
- [ ] The gap is reported as a number, not a picture alone.
- [ ] There is a paragraph on where 2-opt fails.
- [ ] The instance generator is seeded and documented.

**DoP:** "I can implement nearest-neighbour + 2-opt from memory, and explain why 2-opt can be stuck in a local optimum."

**DoDir:** "This feeds the 2027 paper and the logistics research line; it does not feed a frontend career."

---

## 4. Building the review layer

A review is not "looking back". A review is **a decision with a date**. If nothing changes at the end, it was theatre.

### 4.1 Cadence

| Cycle | Time | Work |
|---|---|---|
| Daily | 2 min | Log: intended / done / evidence |
| Weekly | 20 min | Apply the DoD checklist to one artifact; write one line of capability delta |
| Monthly | 45 min | Update **one metric**; record **one decision that changed** |
| Quarterly | 2 h | DoDir: continue / adjust / stop. Re-read the kill criteria |

### 4.2 Five mandatory fields

Structured, because free prose cannot be aggregated — by a human or by an AI.

```
| Date | Intended | Evidence (number/link) | Capability delta | Decision |
```

A real row:

| Date | Intended | Evidence | Capability delta | Decision |
|---|---|---|---|---|
| 2026-11-15 | Write Lab A (NN + 2-opt, gap vs brute force) | commit `a1b2c`, gap 12% at n = 10 | First 2-opt written without copying | Add OR-Tools as a baseline next week |

The two columns most likely to be dropped are **Capability delta** and **Decision**. They are the two columns that turn a diary into a control system.

### 4.3 Three laws

1. **No evidence → not done.** "Studied X" is not evidence. "Built X, link here" is.
2. **Every review must change at least one decision.** No change → reading, not reviewing.
3. **The metric must be outside my control.** If it can be increased by writing more that nobody uses, it is the wrong metric.

---

## 5. Choosing one metric

Selection criteria:

- **Exogenous** — someone else can verify it.
- **Lagging, not leading** — measures outcome, not effort.
- **Unfakeable** — if it can be raised by sitting longer, it is useless.

| Junk metric (currently used) | Real metric (choose one) |
|---|---|
| Number of notes | Number of artifacts used by **a stranger** |
| Hours studied | Number of times someone **reproduced** my result |
| Number of interactions | Number of times I was **reviewed by someone with the right to say no** |
| Books read | Number of things **shipped outside** (merged PR, paper, tool with a user) |

**Chosen metric for this quarter:** *artifacts used by someone I do not know.*

It may legitimately be **zero** this quarter. That is the most honest number available, and it is the only one that forces work out of the room.

---

## 6. The first two-week cycle

**Week 1**

1. Choose **one** artifact — proposed: Lab A (routing baseline).
2. Write a five-question yes/no checklist for it, derived from three reference artifacts.
3. Set the bar at 3/5.

**Week 2**

1. Build the artifact.
2. Apply the checklist by self-marking, then have **one other person** mark it.
3. Write one review row with all five fields.
4. Record **one decision that changed**.

If week 2 produces no changed decision, the review layer does not exist yet. Repeat.

---

## 7. Pitfalls

**1. The first review will feel bad.** It is the first time the number is visible. Discomfort is evidence that it is working, not evidence of incompetence. Most people quit at the first review.

**2. "The artifact failed" will be confused with "I failed".** The standard judges the **artifact**, not the person. If the two collapse, no standard will ever be set — because every miss becomes a self-verdict. This is the same status/self-respect split as in [[06-what-i-want-3-year-self-definition]] §2.2: failing a standard costs *face*, not *self-respect*. Write that sentence at the top of the review file.

**3. The metric will be swapped for a flattering one.** If the chosen metric stays at zero for two quarters, the temptation is to replace it with "notes written". That is the failure mode, not a fix.

**4. Reviews will be skipped in the weeks that matter most.** The weeks with the most work are exactly the weeks with no time to review — and exactly the weeks when the direction should be checked. The 20-minute weekly review is a constraint, not a reward.

**5. Everything will be measured at once.** One metric per quarter. A dashboard with ten numbers is a way of avoiding the one number.

---

## 8. Research Journey

- **Why:** the previous session produced a diagnosis but not an operating procedure. A standard without a review loop is an opinion; a review loop without a standard is a diary.
- **Struggle:** the belief "as long as I keep walking I am not lost" is genuinely protective — it prevents paralysis — so it cannot simply be deleted. It had to be demoted from compass to floor.
- **Aha moment:** the only time series in the repository measures **interactions**. The system was built to measure the exact thing the anti-goals reject. Fixing this is not a matter of discipline but of **changing the variable**.
- **Career link:** an AI Engineer is expected to evaluate models, not just build them — baselines, metrics, ablations, failure analysis. The same discipline applied to one's own work is not a metaphor; it is the same skill. A person who cannot run a review loop on themselves is unlikely to run one on a model in production.

---

## 9. Open questions

- [ ] Which reference artifacts define the standard for the first cycle? Name three, with links.
- [ ] Who is the person with the right to say no for the 2027 paper — a lecturer, a lab supervisor, a reviewer?
- [ ] What is the fallback if "artifacts used by a stranger" stays at zero for two quarters — a new metric, or a new kind of artifact?
- [ ] Does the review layer live in the diary repository, or in a separate private file? *(The diary is public; some review data may not be.)*
- [ ] How does the quarterly DoDir interact with the 3-year kill criteria in [[06-what-i-want-3-year-self-definition]]?

---

## 10. 🎯 Action Items

- [ ] Write the review file header, including: *"The standard judges the artifact, not the person."*
- [ ] Name three reference artifacts and extract five yes/no criteria for Lab A.
- [ ] Set the quarter's bar at 3/5 and the next quarter's at 4/5.
- [ ] Run one review row with all five fields, including one changed decision.
- [ ] Put the monthly review on a fixed calendar date; a review without a date does not happen.

---

## 11. 🔗 Related Topics

- [[06-what-i-want-3-year-self-definition]] — the diagnosis this note operationalises
- [[../random_ideas/07-research-landscape-tech-logistics-traffic]] — the first artifact (Lab A) belongs to this landscape
- [[../books_summaries/07-scientific-research-methodology]] — validity, baselines and evaluation: the same standards applied to research
- [[../ai_ml/05-mlops-lifecycle-and-deployment-architecture]] — drift, evaluation and monitoring: the review loop applied to a model
- [[05-linkedin-research-diary-post-eval]] — an existing example of a review that did produce corrections
- [[../random_ideas/02-time-blocking-method]] — scheduling as the mechanism that makes a cadence possible

### SUGGESTED

- **[SUGGESTED] A personal dashboard note generated from structured fields** — a small table of the quarter's one metric plus the review rows; important because the current calendar measures the wrong variable.
- **[SUGGESTED] Deliberate practice and feedback loops** — `psychology_society/`; the learning-science basis for "drill vs grind" and for why feedback must be external.

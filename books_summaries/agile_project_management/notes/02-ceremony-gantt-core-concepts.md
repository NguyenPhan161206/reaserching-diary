# Session 02 — Ceremonies, Gantt & Core Concepts

**Date:** 2026-09-12
**Source:** [[../knowledge/part-1-understanding-agility.md]], [[../knowledge/part-3-planning-and-execution.md]]
**Tags:** `#Ceremony`, `#GanttChart`, `#Standup`, `#Retrospective`, `#AgileManifesto`

> 📌 This Q&A session originally happened in chat and was lost to a Ctrl+C — content recovered from the opencode database.

---

## ❓ Question 1: What is a ceremony?

**A ceremony** is a *structured, timeboxed* meeting that an Agile team repeats every Sprint. The word "ceremony" sounds formal, but it simply means: **the fixed moments when the whole team gathers to synchronize and improve**.

### 📚 Detailed answer

The 4 main Scrum ceremonies:
| Ceremony | Purpose | Timebox |
|---|---|---|
| Sprint Planning | Pick User Stories for the next sprint | a few hours |
| Daily Standup | Synchronize the next 24h | **15 minutes** |
| Sprint Review | Demo the product to stakeholders | ~1-2 hours |
| Sprint Retrospective | Improve the process | ~1 hour |

Important: ceremonies are **not administrative barriers** — they are the recurring "inspect & adapt" mechanism. If the team meets but improves nothing → that is an empty ceremony (Fake Agile sign #10).

---

## ❓ Question 2: What is Gantt reporting? Why use it? Pros & cons?

### 📚 Detailed answer

#### What it is
A **Gantt chart** is a horizontal bar chart representing a **project schedule**: each task is a horizontal bar, the bar length = estimated duration, the horizontal axis = time, and connected bars show dependencies (task B only starts when task A finishes).

```
Task A ████████████░░░░░░
Task B ░░░░░████████████░  ← depends on A
Task C ░░░░░░░██████░░░░░
        Jan   Feb   Mar
```

#### How to build one
1. List all tasks (Work Breakdown Structure)
2. Estimate the duration of each task (usually in *absolute hours*)
3. Define order and dependencies
4. Draw bars on a timeline
5. Track via % complete (a task "90% done" that stays 90% for 3 weeks 😅)

#### Why people USED to need it (historical root)
Gantt appeared ~1910s and became widespread in **world wars & construction** (Hoover Dam, buildings) — where:
- Requirements are **known and fixed in advance** (blueprints exist)
- The cost of rework is extremely high
- Teams are large and follow orders from above

→ In that world, "plan perfectly up front, then execute exactly" is optimal. Gantt was born to serve that mindset. People still use it because: **everyone understands it, it's visual, and it helps commit to deadlines with clients.** But it assumes that **the world won't change during the project** — true for concrete, false for software.

#### Advantages
- Visual, easy to read, easy to present to leadership
- Shows dependencies clearly → spot blocking tasks
- Good for physical/repetitive work types

#### Disadvantages (for software)
1. **% complete is an illusion** — "task 90% done" doesn't say how much is actually left (in software 90% of UI shown, but 90% of hidden logic untouched)
2. **Assumes change = disaster** — but in dev, change is routine
3. **Encourages dishonest reporting** — nobody wants to report being late, so they "legitimize" nice-looking percentages
4. **Measures no value** — only "how much work was done", not "is the user satisfied"
5. **High maintenance cost** — every schedule change means redrawing the whole board

> **Summary:** Gantt measures **output** (how many tasks are done), while Agile measures **outcome** (does the task create value). That is why Agile replaces "Gantt reporting" with *Information Radiators* (Kanban board, burndown chart) — updated by the team itself, no reporting meetings needed.

---

## ❓ Question 3: What are Standup and Retro?

### 📚 Detailed answer

#### Daily Standup (Daily Scrum)
- **15-minute meeting every morning, standing up** (standing up so it... ends quickly!). Only for the Development Team.
- **3 fixed questions:** What did you do yesterday? What will you do today? Any blockers (impediments)?
- **Its nature:** not a report to a manager — it is **micro-planning for the next 24h**. Find out "is anyone stuck, what does the team need help with".
- ⚠️ If it becomes a "minute-by-minute status report for the boss" → empty ceremony.

#### Retrospective (Sprint Retro)
- **Meeting at the end of the Sprint (~1 hour)**, the whole team reviews **how it works** (not the product — that's Sprint Review's job).
- Familiar framework: **Start / Stop / Continue** — "start doing something new / stop doing something harmful / keep doing what works".
- It is **the most important ceremony** because it's the only place the team directly improves its own process, repeated every sprint → a continuous improvement loop.
- 📌 Deeper: `knowledge/part-3-planning-and-execution.md` Chapter 12.

---

## ❓ Question 4: Does the Project vs Product distinction also show the distinction between the two types of managers?

### 📚 Detailed answer

**Yes, and that's the key point** — but with a refinement: it's not "different kinds of people as managers", but rather **different roles & management culture**:

| | Project Manager (Waterfall) | Product-minded leader (Agile) |
|---|---|---|
| Daily question | "Are we on deadline? % done?" | "Are we solving the right problem?" |
| With the team | Command & control: assign, check, push | Servant-leader: remove blockers, coach, protect the team from distraction |
| With change | Begs for no changes (change = broken plan) | Welcomes it, puts it on the backlog and re-prioritizes |
| Measures | Output: tasks, hours, Gantt % | Outcome: value user receives, velocity, working software |
| With the client | "The contract is sacred, change = lawsuit" | "Partnership, learn together, adapt together" |
| Team | Temporary, borrowed from departments | Fixed long-term, self-organizing |

> Example: A Waterfall Project Manager sees "user changes requirement" as bad news to block. A servant-leader sees it as **free market signal** — the earlier it's detected, the cheaper it is. In short: *Project manager manages the plan; Agile leader manages the user's problem.*

---

## ❓ Question 5: Explain the 4 Values in detail (Part 3)

### 📚 Detailed answer

The Manifesto's intent: **when they conflict, prioritize the left side — but DON'T throw away the right side.**

**① Individuals & interactions > Processes & tools**
- *Meaning:* People and direct communication matter more than applying rigid processes & tools.
- *Example:* A team with state-of-the-art Jira + 200 pages of SOP that never talk to each other → dead. A team with no tools but colleagues sitting next to each other exchanging every hour → alive.
- *Why:* Tools are a means; a "million-dollar tool" can't fix culture.
- *AI connection:* Buying expensive GPUs/autoML won't replace two engineers discussing data.

**② Working software > Comprehensive documentation**
- *Meaning:* A working product must be prioritized over "complete" documentation.
- *Example:* A customer is happier seeing a live demo after 2 weeks than receiving a 300-page spec after 6 months.
- *Why:* Docs are obsolete the moment they're written (because code changes); working software is "the single source of truth".
- *Note:* Minimum docs are still needed (README, API doc) — value 2 doesn't ban docs, it only bans docs being the *reason* to delay delivery.

**③ Customer collaboration > Contract negotiation**
- *Meaning:* Instead of sitting down to "finalize contract terms" to later hold each other to, sit at the same table with the customer continuously.
- *Example:* Traditional contract: "Done within scope = OK, scope change = penalty". Agile collaboration: the customer sits in Sprint Review, sees the product, gives direct feedback → both sides win.
- *Why:* In software products, the customer **cannot know what they need before seeing something running**. Sticking to the contract = guaranteeing both sides lose.

**④ Responding to change > Following a plan**
- *Meaning:* The plan has value, but the ability to react to change has more.
- *Example:* A 6-month plan to build model X. In month 2, you discover there isn't enough data for X, but the data is excellent for Y (smaller, cheaper, users need it more). Agile: pivot to Y. Waterfall: "the plan says X, we build X".
- *Why:* Plans rely on assumptions; wrong assumptions → sticking to the plan means systematically going the wrong way. "Plans are worthless, but planning is invaluable." — Eisenhower.

---

## ❓ Question 6: What is a Hypothesis?

### 📚 Detailed answer

**Hypothesis** = a **falsifiable claim** about the relationship between an action and an outcome, written as "If... Then... Because...". In Agile, it turns product development into science:

```
If   [we build feature X for user group Y]
Then [metric Z will change (up/down)]
Because [the reason we believe that]
```

**Concrete AI example:**
> "IF we add an 'Auto-summarize' button for students cramming for exams, THEN the return rate of the app increases 20%, BECAUSE they spend a lot of time reading long documents."

- The critical point: a hypothesis **must be measurable** (quantifiable metric Z) and **must have an expiry date** ("after 1 sprint, measure…"). Not measurable = not a hypothesis, it's a wish.
- Experiment outcome: **confirmed** (true, scale it up) or **rejected** (you learned something valuable, pivot). Both are wins — because you avoided pouring more $ into something useless.
- This is exactly MVP (Minimum Viable Product): build small enough to *test the hypothesis*, not big enough to "impress".

---

## ❓ Question 7: Where are the 12 Principles in the book / the folder?

### 📚 Detailed answer

**They are in the book** — *Agile Project Management For Dummies* presents the 12 Principles in **Chapter 2 of Part 1**.

In your folder:
- **`knowledge/part-1-understanding-agility.md`** → Chapter 2: short summary: *"The 12 Principles: Emphasize early delivery, welcoming changing requirements, daily business-developer cooperation, and self-organizing teams."* (a one-line summary).
- **`notes/01-agility-mindset-and-manifesto.md`** → the 12 principles grouped into 4 clusters for easy recall (principle numbers noted per cluster).

**The full 12 principles, verbatim** (for you to cross-check — from the Agile Manifesto 2001):

1. Our highest priority is to satisfy the customer through **early and continuous delivery** of valuable software.
2. **Welcome changing requirements**, even late in development — change is a competitive advantage.
3. Deliver working software **frequently**, from a couple of weeks to a couple of months.
4. **Business people and developers work together daily** throughout the project.
5. Build projects around **motivated individuals** — give them the environment and support they need, and trust them.
6. The most efficient and effective method of conveying information is **face-to-face conversation**.
7. **Working software** is the primary measure of progress.
8. Agile processes promote **sustainable development** — maintain a constant, indefinite pace.
9. Continuous attention to **technical excellence and good design** enhances agility.
10. **Simplicity** — the art of maximizing the amount of work *not done* — is essential.
11. The best architectures, requirements, and designs emerge from **self-organizing teams**.
12. At regular intervals, the team reflects on **how to become more effective**, then tunes and adjusts its behavior accordingly.

> Study tip: don't memorize 12 sentences; remember the 4 clusters + that *principles 7 and 8 directly counter the 2 Waterfall sins: measuring progress by "%, hours" and running the team ragged until burnout.*

---

## 💡 Session summary

- [x] Distinguish real ceremonies vs empty ceremonies ("inspect & adapt")
- [x] Understand Gantt chart: advantages (visual, dependencies), disadvantages for software (%, static assumptions, dishonest reporting)
- [x] Standup = 24h micro-planning, Retro = where the team improves process (Start/Stop/Continue)
- [x] Project Manager = manages the plan; Agile leader = manages the user problem
- [x] Deep explanation of the 4 Manifesto values
- [x] Master the "If-Then-Because" hypothesis framework + its link to MVP
- [x] Identify where the 12 Principles live in the book + folder

## 🔄 Suggested next questions

1. [SUGGESTED] **Burndown vs Burnup charts** — the 2 information radiators that replace Gantt, how to read them to know if the team is "heading the right way".
2. [SUGGESTED] **Definition of Done (DoD)** — Part 3 mentions "Done means Done" but doesn't explain why missing DoD is a source of technical debt.
3. [SUGGESTED] **Waterfall vs Agile head-to-head on the same example** — to see exactly when to use which (Agile is not always right!).

---

## 🔗 Related Topics

- [[01-agility-mindset-and-manifesto.md]] — Session 01: mindset, 4 values, 12 principles (4 clusters)
- [[03-scrum-framework-and-four-ceremonies.md]] — Session 03: Scrum, status reports, 4 ceremonies in detail
- [[../knowledge/part-1-understanding-agility.md]]
- [[../knowledge/part-3-planning-and-execution.md]]
- [[README.md]] — Q&A session index

---
*Learn by asking the right questions.*
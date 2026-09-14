# Session 04 — Permanent Team, Tuckman's Phases & Decision Timing

**Date:** 2026-09-12
**Source:** [[../knowledge/part-2-being-agile.md]]
**Tags:** `#MatrixManagement`, `#Tuckman`, `#PermanentTeam`, `#SelfOrganizingTeams`, `#LeanThinking`, `#DecisionTiming`

---

## ❓ Question 1: What is matrix management? Why can't it reach hyper-productivity?

### 📚 Detailed answer

**Matrix management** = the traditional org model where every employee reports to **two managers at once**:
- **Functional manager** (department head, e.g., Head of Engineering) — owns expertise, salary, evaluation
- **Project manager** — owns project-specific work

```
                Project A    Project B    Project C
                (PM A)       (PM B)       (PM C)
Dev 1           ████████
Dev 2                          ████████
Dev 3           ████                     ████
Dev 4                                        ████████
```

Dev 3 spends 50% on Project A and 50% on Project C — one person, two projects, two tug-of-war managers.

#### Why matrix kills hyper-productivity (5 reasons)
1. **Divided loyalty** — 2 managers compete for your time; you never fully "belong" anywhere.
2. **Context switching is expensive** — research (Asana) shows it takes ~**23 minutes** on average to return to a deep-focus state after switching tasks. Working 2 projects = almost never entering "flow".
3. **No team identity** — you never work consistently with the same people, so you never learn each other's working styles or build trust.
4. **Tacit knowledge never accumulates** — every project has a fresh mix of people; "who knows what, who is good at what" is never stored in a stable group.
5. **Never passes Storming** — direct consequence of Tuckman (below) — the team dissolves right when it gets hard.

---

## ❓ Question 2: What are Tuckman's phases?

### 📚 Detailed answer

Bruce Tuckman (1965) defined the lifecycle of a group in 4 phases (Adjourning added later):

| Phase | Name | What happens | Productivity |
|---|---|---|---|
| **1. Forming** | Formation | Polite, probing, unclear roles | Low |
| **2. Storming** | Storm | Conflict, confrontation, power struggle, arguments about direction | Drops → **lowest point** |
| **3. Norming** | Unification | Norms agreed, roles clear, trust begins | Rising |
| **4. Performing** | Peak | High trust, autonomy, mutual correction, works "as one body" | **Hyper-productivity** |

**The key point:** phases take **TIME**. You can't "zoom through".

Pay special attention to **Storming**: many teams fear conflict and avoid it (everyone stays polite) → stuck in Forming forever. Storming is not bad — it's the **price you pay** to reach Performing. Teams should be encouraged to "argue about ideas, not about people."

> **Example:** A new team starts a project: everyone polite (Forming). Week 2, Dev A and Dev B argue about architecture (Storming). If unresolved, the team "freezes" messages (stuck in Storming). If resolved — norms agreed — coordination gets smooth (Norming), and months later they work like an orchestra (Performing).

---

## ❓ Question 3: "Changing teams after every project" — isn't it just "assign as many people as the tech department has to the project"?

### 📚 Detailed answer

This question hits the **core difference**! You just described **matrix management** — and Agile says: **DON'T do that**.

#### Your assumption: "Dept has 20 people, Project X needs 5 → pull any 5 available."
→ That IS traditional (matrix) management. The problem: **every project is a brand-new team of strangers** → always in Forming/Storming → never touches Performing.

#### Agile says: **The Permanent Team** — fixed, long-lived teams

A fixed 5-6 person team, cross-functional (backend, frontend, data, QA), working together on **many consecutive projects/themes over months** instead of dissolving.

Direct comparison:
| | Matrix management | Permanent Team (Agile) |
|---|---|---|
| Members | Assembled from a pool, changes per project | Fixed, long-term |
| Each project | New team, strangers | The same group |
| Tuckman | Forever Forming/Storming | Passes all 4 phases |
| Team memory | Reset each time | Accumulates over time |
| Outcome | Always "average team" | Can reach Performing |

**Easy example:** Pick the 11 best players in the world and assemble them for one match — they will **lose** to a team that has trained together for 5 years — because coordination, mutual understanding, and shared norms matter more than individual skill. Exactly like a dev team.

> **Nuance:** "Fixed" doesn't mean "never change people." It means **change rarely and deliberately**, and when you do, count the "re-Forming cost" — adding 1 person to a 6-person team pushes the whole team **back a step on Tuckman**; the new team needs time to peak again.

---

## ❓ Question 4: Does "Decide as late as possible" mean "decide slowly but surely, act decisively once decided"?

### 📚 Detailed answer

**The "act decisively once decided" part is completely correct.** The "decide slowly" part needs a precise understanding — it's **not procrastination**, it's "keep options open until the **last responsible moment**."

#### Standard definition (from Lean)
> **Defer commitment to the last responsible moment** — delay a decision **until the point where delaying further costs more than deciding right now**.

It is NOT:
- ❌ Lazy procrastination / letting decisions slip away
- ❌ "Wait until the deadline to think"
- ❌ Avoiding hard decisions

It IS:
- ✅ Actively **keeping multiple options open** as long as possible
- ✅ Waiting for the **most important information** to appear before committing
- ✅ Once committed → **execute at full speed** (no hesitation, no second-guessing)

#### 3 criteria to know which decisions "can wait"
| Criterion | Deferable | NOT deferable |
|---|---|---|
| Reversible? | UI color choice | Database architecture (change = $$$$ ) |
| More info if we wait? | Pick framework once requirements known | Legal deadline (hard date) |
| Wait-cost < change-cost? | Choose model after data arrives | Safety / security / no-return decisions |

#### Concrete AI example (for you)
> Don't decide "model architecture X, GPU Y" in week 1 when you haven't seen **real data**. Keep the data pipeline and model interface **flexible** (swappable), wait until data + metric baseline exist → **then** commit to architecture → then decisively train, deploy, scale.
> On the flip side, don't defer: illegally scraping data without checking copyright, or paying salaries — those can't wait.

#### Memory aid
> **"Slow to decide, fast to act."** — Deliberately slow at the decision stage to gather the most information. Absolutely fast at execution once committed. Deferring a decision ≠ deferring action.

---

## 💡 Session summary

- [x] Understand matrix management (2 managers, multi-project splitting) and its 5 downsides
- [x] Master Tuckman's 4 phases (Forming → Storming → Norming → Performing) + why Storming is necessary
- [x] Contrast matrix "pool assembly" vs Agile "permanent team" — team memory accumulates, re-Forming costs
- [x] "Decide as late as possible" = defer to last responsible moment (not procrastinate) + act decisively after

## 🔄 Suggested next questions

1. [SUGGESTED] **How leaders steer each Tuckman phase** — what should a servant-leader do differently in Forming vs Storming vs Performing
2. [SUGGESTED] **Option-based thinking** — concrete technique to "keep options open" in software architecture (deferred decisions, feature flags)
3. [SUGGESTED] **ScrumBan (Scrum + Kanban)** — how permanent teams combine frameworks in practice

---

## 🔗 Related Topics

- [[../knowledge/part-2-being-agile.md]] — source: Chapter 7-8 roles & permanent team, Chapter 5 Lean
- [[02-ceremony-gantt-core-concepts.md]] — Session 02: project vs product managers (servant-leader context)
- [[03-scrum-framework-and-four-ceremonies.md]] — Session 03: Scrum roles context
- [[ai_ml/04-xp-engineering-practices-for-ml.md]] — team/CI context in engineering practice
- [[README.md]] — Q&A session index

---
*Learn by asking the right questions.*
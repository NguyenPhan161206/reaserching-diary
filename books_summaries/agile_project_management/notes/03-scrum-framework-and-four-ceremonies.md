# Session 03 — The Scrum Framework & The 4 Ceremonies in Detail

**Date:** 2026-09-12
**Source:** [[../knowledge/part-2-being-agile.md]], [[../knowledge/part-3-planning-and-execution.md]]
**Tags:** `#Scrum`, `#DailyStandup`, `#SprintPlanning`, `#SprintReview`, `#SprintRetrospective`, `#StatusReport`

> 📌 This Q&A session originally happened in chat and was lost to a Ctrl+C — content recovered from the opencode database.

---

## ❓ Question 1: What is Scrum?

### 📚 Detailed answer

**Scrum** is the most popular **framework** (not a rigid process) in Agile, used to manage complex knowledge work. It's a "skeleton" you fill with your own team.

Scrum rests on **3 pillars**:
- **Transparency**: everything is visible (backlog, Kanban board, burndown)
- **Inspection**: frequent checks (Standup daily, Review at sprint end)
- **Adaptation**: adjust immediately on detecting drift (Retro every sprint)

**Structure of 1 Sprint:**
```
Sprint (1-4 weeks, timeboxed)
├── Sprint Planning      → pick User Stories, set the goal
├── Daily Standup (every day) → synchronize 24h
├── Sprint Review        → demo to stakeholders, get feedback
└── Sprint Retrospective → the team improves its process
```

**3 roles:**
| Role | Owns | Responsibility |
|---|---|---|
| Product Owner (PO) | "What" | Optimizes backlog value, decides priorities, says "no" to stakeholders |
| Scrum Master (SM) | "Process" | Servant-leader: removes impediments, coaches, does not manage people |
| Development Team | "How" | Self-organizing, self-assigns work, cross-functional |

**Artifacts** (management products): Product Backlog (prioritized list), Sprint Backlog (this sprint's work), Increment (the "Done" product increment).

📌 Sources in the folder: `knowledge/part-2-being-agile.md` (Chapter 5, the Scrum section) and `knowledge/part-3-planning-and-execution.md` (ceremonies).

---

## ❓ Question 2: What is a "status report"?

### 📚 Detailed answer

**Status report** = traditional status reporting: each person/week lists *"What I did, how far along I am, in % vs the plan"* for superiors to control.

**Its problem in Agile:**
- A Standup is NOT a status report to the boss. If the team meets but speaks with a "the boss is listening" attitude → it becomes a "tick-tock status report" — a waste of 15 minutes that creates a defensive culture (everyone brags about achievements and hides difficulties).
- A proper Standup is **the team talking to each other**: "I'm stuck here, can anyone help?" → impediments are cleared immediately, no need to wait for a weekly report.

> Rule of thumb: *Status report = information going UP so someone can control. Standup = information going ACROSS so the team can coordinate itself.*

---

## ❓ Question 3: Explain the 4 Scrum ceremonies in detail

### 📚 Detailed answer

A standard sprint has **4 events** (the Scrum Guide calls the Sprint a "container" holding all 4):

### 🔷 A. Sprint Planning (start of sprint)
- **Who:** PO + Development Team (+ SM assists).
- **Timebox:** ~2 hours per week of sprint (2-week sprint → ~4-hour meeting).
- **Answers 2 questions:** *"What will we do (Which)?"* and *"How will we do it (How)?"*
- **Flow:**
  1. PO presents the highest-priority User Stories from the Product Backlog, clearly stating the **Sprint Goal**.
  2. The team picks stories it can commit to in the available time (based on **Velocity**).
  3. The team splits tasks itself (don't let the manager assign!) and estimates.
- **Why it matters:** it's the only place the team "commits" to the sprint scope — skip it and the sprint has no clear starting point.

> **Example:** Sprint Goal: *"New users can sign up with Google within 1 minute."* → Stories: "OAuth login", "Default profile creation", "Welcome email". The team has estimated 2 stories; story 3 is unclear → bring it to the table and agree to drop it this round to guarantee "Done".

### 🔷 B. Daily Standup / Daily Scrum (EVERY day of the sprint)
- **Who:** Development Team only (PO/SM join optionally, not required). SM may not even be needed.
- **Timebox:** **15 minutes. Standing.** On time, even when late.
- **3 questions:**
  1. What did I do yesterday (contributing to the Sprint Goal)?
  2. What will I do today?
  3. Are there any impediments blocking me?
- **It is not**: reporting to the boss, nor solving problems right in the meeting (deep work → breakout after the meeting).
- **Why it matters:** it's the 24h "heartbeat" — enables micro-inspection: if someone drifts off course, the team detects it in a day instead of at the end of the week.

> **Example:** "Yesterday I finished the login API. Today I'm building the profile page. Blocker: no test credentials from the backend team yet — can someone provide them?" → Another dev: "I'll issue them in 10 minutes." Problem dies right in the meeting.

### 🔷 C. Sprint Review (end of sprint — product focus)
- **Who:** Team + **stakeholders/customers** (mandatory!) + PO.
- **Timebox:** ~1 hour per week of sprint (2-week sprint → ~2 hours).
- **Content:**
  1. The team demos **working software** (not slides!) — whatever is "Done".
  2. Stakeholders look, try, and give feedback.
  3. PO updates the Product Backlog based on feedback (add/change/re-prioritize).
- **Why it matters:** it's the **earliest possible feedback mechanism** — "building the wrong thing" is the deadliest risk; Review is the vaccine against it. This is where the customer "sits at the same table" per Manifesto Value #3.

> **Example:** After demoing the "Google sign-up" feature, the customer: "Oh nice, but we also need phone-number login." → PO writes a new story, high priority in the backlog → the team does it next sprint. Without a Review, the team would only find out the customer needs something else after finishing — costing a whole sprint.

### 🔷 D. Sprint Retrospective (end of sprint — process focus)
- **Who:** Team only (NO stakeholders). SM facilitates.
- **Timebox:** ~45-60 minutes.
- **Content:** review HOW the team worked in the past sprint. Classic framework: **Start / Stop / Continue**:
  - *Start:* new things to do (e.g., add code review)
  - *Stop:* harmful habits (e.g., committing straight to main)
  - *Continue:* things to keep (e.g., effective pair programming)
- **Most important:** pick **1-3 SPECIFIC action items with an owner** to implement immediately next sprint. A Retro without action items = a pointless meeting.
- **Why it matters:** it's the **Adaptation** loop in the cycle — the only place the team upgrades its way of working. "Work smarter" happens here, not at the code.

> **Example:** The team notices that in the past sprint, 3 deadlines slipped because stories were "done" but broke in production. → Action item: *"From now on, a story is only Done when it passes CI and runs on staging — owner: Minh, deadline: before next Sprint Planning."*

---

## 🔄 The 4 ceremonies in 4 sentences

| Ceremony | Central question | Focus |
|---|---|---|
| Sprint Planning | *"What do we do, and what next?"* | Decision |
| Daily Standup | *"Next 24h, who needs help?"* | Synchronization |
| Sprint Review | *"Is this product what the customer wanted?"* | Product + feedback |
| Retrospective | *"Is our way of working getting better?"* | Process |

> 💡 Memory trick: **Planning looks forward, Review looks at the product, Retro looks at the team itself. Standup keeps all three on track every day.**

---

## 💡 Session summary

- [x] Understand Scrum = a framework with 3 pillars (Transparency, Inspection, Adaptation)
- [x] Master the 3 roles (PO/SM/Dev Team) + artifacts (Product/Sprint Backlog, Increment)
- [x] Distinguish Standup (ACROSS, coordination) vs Status report (UP, control)
- [x] Explain all 4 ceremonies in detail: who, timebox, flow, why each matters
- [x] A Retro must end with concrete action items + owners — otherwise it's a pointless meeting

## 🔄 Suggested next questions

1. [SUGGESTED] **Velocity & Story Points** — Sprint Planning "picks how many stories" based on it; without understanding velocity, planning becomes guesswork.
2. [SUGGESTED] **Definition of Done (DoD)** — what decides whether a story counts as "Done" in Review.
3. [SUGGESTED] **Burndown chart** — the information radiator that measures daily whether the team will hit the sprint goal.

---

## 🔗 Related Topics

- [[01-agility-mindset-and-manifesto.md]] — Session 01: mindset, Manifesto
- [[02-ceremony-gantt-core-concepts.md]] — Session 02: ceremonies overview, Gantt, hypothesis
- [[../knowledge/part-2-being-agile.md]]
- [[../knowledge/part-3-planning-and-execution.md]]
- [[README.md]] — Q&A session index

---
*Learn by asking the right questions.*
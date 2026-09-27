# Building a Second Brain — Tiago Forte

**Created Date:** 2026-09-28
**Tags:** `#Productivity`, `#KnowledgeManagement`, `#NoteTaking`, `#PARA`, `#PersonalDevelopment`
**Rating:** ⭐⭐⭐⭐☆ (4/5 — see *Honesty note* below)
**Genre:** Productivity & Self-Management
**Status:** **Framework Reference** — *not* a first-hand full read. Written from the published framework and secondary sources, to anchor the system this diary already uses. Must be replaced with a real read-through summary once the book is finished.

**Honesty note:** the template for book notes asks for chapter-by-chapter analysis and personal reflection on reading. That would be fabricated here. What follows is an accurate map of the framework plus an honest assessment of how much of it this repository already implements — which is the actual reason for writing it.

---

## 📌 Key Takeaways (TL;DR)

- A **second brain** is not a storage system; it is a **funnel + habit workflow** that moves information from fleeting to permanent to useful in three states.
- The whole system is **one principle applied twice**: externalise everything, then process everything on a daily cadence.
- **PARA** (Projects, Areas, Resources, Archives) organises by **actionability**, not by subject. Subject-based folders fail; actionability-based ones survive.
- **Progressive summarization** is the reason daily skimming works: you are not re-reading, you are *re-compressing* toward an executive summary, one layer at a time.
- The unit of capture is **one note per idea**, which makes notes atomic and therefore linkable — the property that turns a folder of files into a network.

---

## 📖 Book Overview

- **Author background:** Tiago Forte, consultant and founder of *Forte Labs*, creator of the Progressive Summarization method and the PARA method. He built the system publicly as a working productivity experiment, which is why the book's most useful material is documented in public artefacts as well.
- **Publication year:** 2022 (Portfolio/Penguin, expanded from earlier work on Progressive Summarization).
- **Core thesis:** Your brain is optimised for holding a very small amount of information for a very short time. The solution is not to try harder — it is to move the storage burden outside yourself and then **process what you have externalised on a strict daily habit**.
- **Target audience:** knowledge workers, researchers, writers, students — anyone who reads more than they can hold. Readable by an IT student; no technical prerequisite.

---

## 🧠 Detailed Analysis

### 1. The CODE acronym

Forte's central mental model, and the reason the book is worth reading: **you are the CODE** — the running program — and the external system is your memory.

| Letter | Role | Concrete in this repository |
|---|---|---|
| **C** — Capture | Siphon anything potentially useful, cheaply, at the moment of noticing | `inbox/`, committing lecture notes, adding a term to `vocabulary/` |
| **O** — Organize | File each item into one actionable category | Folder structure by domain (`ai_ml/`, `vocabulary/`, `dev_foundation/`) |
| **D** — Distill | Compress toward what matters, progressively | Note template selection, section-by-section depth, "Viet-Glish pitfalls" tables |
| **E** — Express | Produce something in your own voice, in the output medium | LinkedIn post, CV, GitHub commit messages |

**Why this matters here:** every folder in this repository maps to one of the four stages, and the practical failure mode of such a system is skipping **Distill** — capturing and organising forever, never compressing. That failure produces a large, well-organised archive nobody re-reads, which is exactly the fate of most "second brain" content online.

### 2. PARA — organise by actionability, not by subject

| Category | Definition | Test question | In this repository |
|---|---|---|---|
| **Projects** | a goal with a **deadline** | *Will this be done by a date?* | a specific deliverable — e.g. the Joblink release |
| **Areas** | a standard to **maintain** forever | *Is this ongoing?* | `ai_ml/`, `dev_foundation/`, `cv/`, English ability |
| **Resources** | a topic of **interest** | *Would I ever want to consult this?* | `vocabulary/`, `books_summaries/`, reference notes |
| **Archives** | **inactive** items from the other three | *Is this done?* | superseded notes, old CV versions |

**The argument against subject folders:** a note about MQTT belongs simultaneously to *IoT*, *networking*, *debugging* and *my current project*. Filed by subject, it can only live in one place, and the other three become broken links. Filed by actionability, it lives with whatever you are currently doing, and a single note can still carry multiple subject tags. **This repository is PARA-adjacent, not PARA-compliant**: the folders are subject-based (domains of knowledge), which works for a *knowledge base* but would break as a *project tracker*.

### 3. Progressive Summarization — the daily skim, named

This is the single most useful idea in the book, because it gives a name to a habit the author has been doing intuitively: skimming yesterday's notes.

Every item passes through five layers of compression, and each layer is reached only when the item is *useful again*:

| Layer | Action | Applied to a lecture note |
|---|---|---|
| 1. **Capture** | store it whole | every sentence from the lecture |
| 2. **Bold** | bold the sentences that carry the argument | the 5 key claims |
| 3. **Highlight** | highlight the best parts of the bolded text | the 2 strongest claims |
| 4. **Summary** | write a short summary in your own words | a 4-sentence TL;DR |
| 5. **Remix** | combine with other notes into something new | a LinkedIn post, a design decision |

**The counter-intuitive rule: you are not supposed to summarise on day one.** A note is compressed only when you encounter it again. This inverts the usual instinct to "write it properly now", and it is why progressive summarization scales to years while "write perfect notes immediately" collapses under its own weight.

**The economics:** a note read 3 times ends up roughly 10× shorter than the one you captured. A note never read again costs only the capture effort, which is designed to be seconds. The system is therefore **cheap for unused items and cheap to re-enter for used ones** — an asymmetric cost profile, which is the actual design insight.

### 4. The daily habits workflow

The book is explicit that the system fails without a **routine**:

| Habit | What it does | Why it cannot be skipped |
|---|---|---|
| **Capture** | one inbox, capture anywhere, review once a day | Backlog is what kills the system |
| **Distill** | process today's inbox to empty, every day | *"An inbox processed daily stays small"* |
| **Calendar** | a schedule of commitments, reviewed weekly | Separates *what you must do* from *what you want to learn* |

**The rule with the highest leverage: never let the capture inbox grow overnight.** This repository reproduces it structurally — `inbox/` exists as a staging area, `CONTRIBUTION_LOG.md` records every touch, and `CONTRIBUTION_CALENDAR.md` tracks the streak so the habit is measurable.

### 5. What the book gets right, and this diary already does

| Book's prescription | Status in this repository |
|---|---|
| One external system, not five | ✅ Single Markdown tree in one Git repository |
| Plain text as the format | ✅ Markdown everywhere, no proprietary database |
| Version history | ✅ Git — full history, diff, revert |
| Actionable organisation | ⚠️ Subject-based folders instead of PARA |
| Progressive summarization | ⚠️ Implicit — no per-note layers, no deliberate Layer 4 |
| Daily capture/distill routine | ✅ `CONTRIBUTION_LOG.md` + calendar + streak |
| Express stage | ⚠️ Weak — 12 notes, very little output *derived* from them |
| Portable and future-proof | ✅ Markdown survives any tool; this is the claim the LinkedIn post should make |

**The honest verdict:** this repository already implements roughly **60% of the method** — and the two missing 20% blocks (progressive summarization, and the Express stage) are precisely the two that produce visible results to other people. That is why the LinkedIn post felt thin: it described the *plumbing* rather than the *output*.

---

### 6. The lineage the book does not cover: Zettelkasten

Forte's system organises *one person's* notes. The other major lineage, **Zettelkasten**, organises notes for *thinking*:

| | Forte's second brain | Zettelkasten |
|---|---|---|
| **Origin** | Tiago Forte, 2022 | Niklas Luhmann's paper system; formalised by Sönke Ahrens (*How to Take Smart Notes*, 2017) |
| **Unit** | a project, area, resource or archive | **one atomic idea per note** |
| **Linking** | by folder and tag | by **explicit links between ideas** (`[[…]]`) |
| **Goal** | fast recall of what you have collected | **emergent thinking** — new ideas generated by linking |
| **Failure mode** | a well-organised archive nobody re-reads | a dense, unreadable web with no entry point |

**Why the pair belongs in the same note:** this repository already uses `[[wiki-links]]` between notes — the Zettelkasten half — inside PARA-less, subject-based folders — the Forte half. Neither half is complete alone. A folder tree gives you an **entry point**; atomic linked notes give you **new combinations**. The synthesis is the actual goal, and it is not a book recommendation so much as an unfinished design problem for this diary.

---

## 💡 Profound examples (from the framework, not from the text)

> ⚠️ *Because this is a framework reference, these are worked applications to this repository, not illustrations quoted from the book. They are marked as such deliberately.*

**Example 1 — one note, two homes**
A note about MQTT Last Will and Testament is simultaneously *Material* (broker design) and *Skill* (examining broker internals). Filed by subject it breaks one link. Tagged `#mqtt #iot #distributed-systems` and linked from both the MQTT note and the LWT note, it stays reachable from either direction. **Why it is powerful:** it exposes that the *unit* of organisation in a knowledge system is the question, not the topic.

**Example 2 — why a research diary is cognitive offloading, formally**
Risko & Gilbert (2016) define *cognitive offloading* as using physical action to enhance cognitive performance. A lecture note is offloading. So is a sticky note, a calendar reminder, and a commit message. **Why it is powerful:** it makes the author's LinkedIn post citable instead of merely personal, and it puts the diary in a research literature that the author can then read on purpose.

**Example 3 — the tool-switching question gets a framework**
The author's Notion → GitHub move is usually defended on price or AI features. Forte's actual criterion is **format ownership**: a portable format cannot be held hostage. Markdown in Git with a full history is portable; a Notion export is a lossy copy of someone else's database. **Why it is powerful:** the weaker argument (context limits) expires when the competitor ships a feature; the stronger argument (ownership) does not.

---

## 🔗 Cross-Book Connections

- **Forte ↔ Luhmann / Ahrens (*How to Take Smart Notes*)** — capture and organise vs. link and generate; the two halves described in §6
- **Forte ↔ *Deep Work* (Cal Newport, `books_summaries/03`)** — both prescribe a **shutdown ritual**; Forte adds the external store, Newport adds the attention budget. Deep Work is about *not* being interrupted; Forte is about *not* losing what you already read
- **Forte ↔ *Atomic Habits* (Clear, `books_summaries/04`)** — the daily habits workflow is an *identity-based habit* ("I am someone who processes his inbox daily"), which is exactly the habit-stacking mechanism Clear describes. The calendar and streak in this repository are the visible evidence loop
- **Forte ↔ *Thinking, Fast and Slow* (Kahneman, `books_summaries/02`)** — System 1 forgets; the whole enterprise exists to compensate for a known cognitive limitation. Forte is Kahneman applied as infrastructure
- **Forte ↔ *Agile Project Management For Dummies* (`books_summaries/05`)** — Forte's Projects-with-a-deadline is the Agile sprint; "distill to empty" is the *Definition of Done* applied to personal knowledge instead of a product backlog

---

## 🧠 How to Apply

**Action items, in order of leverage**

1. **Adopt one progressive-summarization habit for existing notes.** Take the 5 most-read notes in this repository and add a Layer 4 summary (a TL;DR in your own words) at the top. Cost: one pass. Effect: the notes become re-enterable in 30 seconds instead of 5 minutes.
2. **Split "Areas" out of the subject tree.** `dev_foundation/` is an *Area* (ongoing, no end date); `books_summaries/06-…/` is a *Project* until it is finished. Naming which is which prevents a folder from being abandoned when its topic goes quiet.
3. **Add a `## 💡 My Synthesis` section to every note.** Forte's Layer 5 (Remix) is the only stage this repository does not perform. One paragraph per note that connects it to something else is enough to start generating new material.
4. **Build the Express stage deliberately.** One LinkedIn post, one blog post, or one conference talk per month, derived from the notes. Twelve notes that produce no output is a private archive, not a second brain.
5. **Keep the daily capture/distill rule.** `inbox/` → processed → moved, same day. The calendar and log already enforce it; do not let the rule relax.
6. **One note per idea for anything that will be *linked* later.** Long notes are hard to link; atomic notes are easy. Long-form notes can stay in the folder, but anything you expect to connect to something else should be atomic.

**Career application (AI Engineer context)**

- **Portability is a professional requirement.** Model checkpoints, configs and research notes are all *format ownership* decisions. Choosing Markdown over a proprietary note app is the same decision as choosing a portable checkpoint format over a vendor-locked one.
- **Progressive summarization is a technique for reading ML papers** — read for the abstract and figures first (Layer 3), write your own summary (Layer 4), then link it to the paper it contradicts (Layer 5). This is faster and more durable than reading every paper once, end to end.
- **The Express stage is a portfolio.** A public artefact per month, derived from the notes, is a demonstrated record of technical synthesis. It is the difference between "I read a lot" and "I can show what I understood".
- **Version control as external memory** is a transferable professional skill, not a personal quirk: the habit of reviewing a diff, reverting safely, and keeping a legible history transfers directly to code review, experiment tracking and ML model versioning.

---

## 📝 Research Journey

- **Why:** The LinkedIn post about the research diary prompted the question *"does anyone already have a name for this?"* — and two people do: Forte (the system) and Luhmann (the linking). Knowing the lineage turns a personal hack into a method with a literature, which matters for a career built on writing.
- **Biggest surprise:** **progressive summarization does not require summarising on capture.** The instinct to write a good note immediately is the thing that makes note-taking fail; the method only compresses an item when it is useful *again*. That single reframe makes lifelong note-taking affordable.
- **Changed my mind about:** folder structure. Subject-based folders feel tidy and were assumed to be the right organising principle. They are only right while the library is small; PARA's actionability test is the better default, and the reason this repository's folders eventually stop working is that a note about MQTT belongs to four subjects at once.
- **Will implement:** a Layer 4 TL;DR on the 5 most-read notes; one `## 💡 My Synthesis` section per new note; one public artefact per month derived from the notes.

---

## 🧪 Practice & Self-Assessment

1. Place each of these in PARA: *(a) a job application with a deadline, (b) English vocabulary, (c) an old CV version, (d) a hobby project with no deadline.*
2. A note you captured 8 months ago and have not opened since. Which layer of progressive summarization is it on? What is the cheapest useful action on it now?
3. Explain the difference between Forte's second brain and Zettelkasten in one sentence each, and name one thing each can do that the other cannot.
4. Why does "write perfect notes immediately" collapse, while "capture fast, compress on reuse" scales? Answer in terms of cost asymmetry.
5. **A trap question:** a note about a *specific* lesson belongs in Projects or in Areas? Defend your answer.

---

## 🔗 Related Topics

- [[books_summaries/05-agile_project_management/README.md]] — Projects-with-a-deadline is the same idea as a sprint; "distill to empty" is a Definition of Done
- [[books_summaries/03-deep-work.md]] — the attention budget that makes the daily workflow necessary
- [[books_summaries/04-atomic-habits.md]] — the habit-stacking mechanism behind "distill every day"
- [[vocabulary/12-one-vietnamese-word-many-english-words.md]] — the Express stage in practice: choosing the register that makes a technical idea readable
- [[personal_evaluations/05-linkedin-research-diary-post-eval.md]] — the artefact produced by the Express stage
- [SUGGESTED] **Sönke Ahrens — *How to Take Smart Notes* (2017)** — *why important:* it is the formalisation of the linking half, and the failure mode (no entry point) is the one this repository's folder tree is currently risking. *Folder:* `books_summaries/`
- [SUGGESTED] **Ebbinghaus / spaced repetition — the review cadence behind daily skimming** — *why important:* "day after day" review is currently justified only by habit; the forgetting curve would justify it with evidence. *Folder:* `psychology_society/`
- [SUGGESTED] **A `## 💡 My Synthesis` section added to the existing NOTE_TEMPLATE** — *why important:* the template currently produces good captures and no synthesis, which is the gap §3 identifies. *Folder:* config change — requires approval before editing `NOTE_TEMPLATE.md`

---

## 📚 Source Note

- **Framework accuracy:** PARA and progressive summarization are Forte's published public methods and are documented at `fortelabs.com`. The five-layer compression model, the CODE acronym and the daily habits workflow are the framework's standard components.
- **Not claimed from the text:** chapter-by-chapter analysis, page references, and quoted examples. None of that appears here because the book has not been read in full. Treat the "Profound examples" section as original application, not summary.
- **To upgrade this note:** read the book, replace the Status line, add a real per-chapter breakdown and a rating with justification, and keep the PARA-versus-Zettelkasten analysis in §6 as the genuinely original contribution.

---
*Books are seeds. Notes are the garden. This one is a seed catalogue, not a harvest — and it says so.*

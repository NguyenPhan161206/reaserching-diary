# DSA — Anchor & Roadmap (code lives in a separate repo)

> **This folder is an anchor, not a container for solutions.**
> Algorithm code, per-problem explanations, and the practice log live in a dedicated repository so that the code has a clean, fast-to-clone structure — while this diary keeps the *concepts, roadmap, and reflections*.

## 🔗 Code repository

**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 🎯 The division of labour (keep it clean)

| Lives in `reaserching-diary` (this repo) | Lives in `DSA-reserching-diary` (code repo) |
|------------------------------------------|--------------------------------------------|
| Learning roadmap and progress tracking | Solution source code (`solution.java`, `solution.cpp`) |
| Language notes (how Java and C++ differ for DSA) | Per-problem `approach.md` write-ups |
| Cross-technology notes (how the CLI, Git, complexity tie together) | Practice logs, contest results, error journal |
| Mental models and *aha moments* | Local experiments and benchmarks |
| Pointers to problems worth studying | Committed benchmark results |

**Rule:** no solution code is duplicated into this repo. Notes here **point** (AGENTS.md Section L).

---

## 🎯 The two-track language split

This track uses **two languages, each with a different job**. JavaScript is deliberately *not* here — it belongs to the web track.

| Track | Language | Role | Where the notes live |
|-------|----------|------|-----------------------|
| **DSA** | **Java** | Course language — every problem attempted here first | [02-java-for-dsa.md](02-java-for-dsa.md) |
| **DSA** | **C++** | Port target — interview fluency, contests, tight constant factors | [03-cpp-for-dsa.md](03-cpp-for-dsa.md) |
| **Web** | JavaScript | The web track (DOM, event loop, async) | [`../web_basics/`](../web_basics/README.md) |
| **AI/ML** | Python | The ML stack (Snake_ANN, notebooks) | [`../../ai_ml/`](../../ai_ml/README.md) |

**Why two, not one:** attempting in the language you are graded in (Java) removes a translation step under time pressure; porting to C++ surfaces which parts of a solution are language-specific rather than algorithmic. Porting **from memory** and writing a **Language Notes** section in the solution is what makes the second language pay off — a line-by-line transliteration teaches nothing.

**Rule:** attempt in Java → port to C++ only for problems where C++'s tooling is a genuine advantage (heaps, number theory) or for interview prep. Do not solve everything twice from scratch.

---

## 📋 Note Index

| # | Note | Purpose |
|---|------|---------|
| 01 | [DSA Learning Roadmap](01-learning-roadmap.md) | Ordered plan, progress table, exit criteria per phase |
| 02 | [Java for DSA](02-java-for-dsa.md) | Course language: collections, generics, overflow, `StackOverflowError` |
| 03 | [C++ for DSA](03-cpp-for-dsa.md) | Port target: STL, value semantics, iterator invalidation, UB |
| 04 | [Complexity Analysis Toolkit](04-complexity-analysis-toolkit.md) | $O$ / $\Theta$ / amortised / recurrences, when Big-O lies, and how to measure honestly |
| 05 | [Data Structure Selection Guide](05-data-structure-selection-guide.md) | Operations inventory → structure; the invertibility/associativity/idempotence rule; what Java and C++ do **not** give you |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Recursion → iteration mental model | Bridges tree/graph traversal and both languages' stack limits (referenced by 02, 04, 05) |
| Medium | How to read a problem statement like a spec | Input/output/constraints first, examples last — the prerequisite for the inventory in 05 |
| Medium | Java ↔ C++ porting decision guide (when the second language pays off) | Turns the 1-in-5 rule into a checklist; the availability table in 05 is the raw material |
| Medium | Common bug patterns in competitive programming | Off-by-one, overflow, wrong base case — overlaps the Pitfalls tables in 02–05 |
| Low | Interview vs contest mindset difference | Only when job hunting starts |
| Low | `07-probabilistic-methods.md` (expected vs amortised, hashing, Monte Carlo, birthday paradox) | Referenced from 04; the theory behind "expected $O(1)$" |

---

## 🔗 Tech Cross-References (Section K)
- **Java / C++** — the two languages of this track → [02-java-for-dsa.md](02-java-for-dsa.md), [03-cpp-for-dsa.md](03-cpp-for-dsa.md)
- **Linux** — the CLI is how you run, profile, and debug algorithm code (`g++`, `time`, `| less`, `valgrind`) → [`../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md), [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md)
- **JavaScript** — the *web* track language, deliberately outside this one → [`../web_basics/README.md`](../web_basics/README.md)
- **Python** — the AI/ML language, not a DSA language here → [`../../ai_ml/README.md`](../../ai_ml/README.md)
- **Git** — every solution is a commit; branches per topic make the history a study timeline
- **Databases** — B-tree indexes, hash maps, and query planners *are* the data structures from this track
- **Big-O ↔ scaling economics** — the same trade-off curve appears in business (marginal cost, economies of scale) → see `economics_business/`
- **ML** — algorithmic thinking transfers to gradient descent, sampling, and complexity of model training
- **Docker** — a reproducible environment for competitive programming judges and stress testing


## 🔗 Related Topics
- [`../linux/README.md`](../linux/README.md) — the tool you use while doing DSA
- [`../../git_github/README.md`](../../git_github/README.md) — prerequisite track; Git is how DSA progress is recorded
- [`../databases/README.md`](../databases/README.md) — data structures with a storage cost model
- [`../web_basics/README.md`](../web_basics/README.md) — the other track (JavaScript)
- [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md) — scaling curves, same mental shape as complexity
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `bridge-complexity-and-scaling.md` — bridge note: $O(n)$ in code vs economies of scale in business

---

**Reading order for this folder:** 01 (the plan) → 04 (the cost model) → 05 (the selection method) → 02/03 (the two languages). The two language notes are usable immediately, but 04 and 05 are what make the difference between recognising a problem and solving it.

---
*Created: 2026-09-28 — Template: Flexible (anchor + roadmap). No solution code in this repo (AGENTS.md Section L).*

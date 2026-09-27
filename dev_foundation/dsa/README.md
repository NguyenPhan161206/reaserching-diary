# DSA — Anchor & Roadmap (code lives in a separate repo)

> **This folder is an anchor, not a container for solutions.**
> Algorithm code, per-problem explanations, and the practice log live in a dedicated repository so that the code has a clean, fast-to-clone structure — while this diary keeps the *concepts, roadmap, and reflections*.

## 🔗 Code repository

**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 🎯 The division of labour (keep it clean)

| Lives in `reaserching-diary` (this repo) | Lives in `DSA-reserching-diary` (code repo) |
|------------------------------------------|--------------------------------------------|
| Learning roadmap and progress tracking | Solution source code (C++ / Python) |
| Cross-technology notes (how the CLI, Git, complexity tie together) | Per-problem `approach.md` write-ups |
| Mental models and *aha moments* | Practice logs, contest results, error journal |
| Pointers to problems worth studying | Local experiments and benchmarks |

**Rule:** no solution code is duplicated into this repo. Notes here **point** (AGENTS.md Section L).

---

## 📋 Note Index

| # | Note | Purpose |
|---|------|---------|
| 01 | [DSA Learning Roadmap](01-learning-roadmap.md) | Ordered plan, progress table, exit criteria per phase |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Complexity analysis toolkit ($O$, amortized, space, when to trust it) | Every other DSA decision depends on it |
| High | Data-structure selection guide (which structure for which problem shape) | The skill that separates "memorised solution" from "derived solution" |
| Medium | Recursion → iteration mental model | Bridges to tree/graph traversal and to JS call stacks |
| Medium | How to read a problem statement like a spec | Input/output/constraints first, examples last |
| Low | Common bug patterns in competitive programming | Off-by-one, overflow, wrong base case — write once, review often |
| Low | Interview vs contest mindset difference | Only when job hunting starts |

---

## 🔗 Tech Cross-References (Section K)
- **Linux** — the CLI is how you run, profile, and debug algorithm code (`g++`, `time`, `| less`, `valgrind`)
- **Git** — every solution is a commit; branches per topic make the history a study timeline
- **Databases** — B-tree indexes, hash maps, and query planners *are* the data structures from this track
- **Big-O ↔ scaling economics** — the same trade-off curve appears in business (marginal cost, economies of scale) → see `economics_business/`
- **ML** — algorithmic thinking transfers to gradient descent, sampling, and complexity of model training
- **Docker** — a reproducible environment for competitive programming judges and stress testing

## 🔗 Related Topics
- [`../linux/README.md`](../linux/README.md) — the tool you use while doing DSA
- [`../../git_github/README.md`](../../git_github/README.md) — prerequisite track; Git is how DSA progress is recorded
- [`../databases/README.md`](../databases/README.md) — data structures with a storage cost model
- [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md) — scaling curves, same mental shape as complexity
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `bridge-complexity-and-scaling.md` — bridge note: $O(n)$ in code vs economies of scale in business

---
*Created: 2026-09-28 — Template: Flexible (anchor + roadmap). No solution code in this repo (AGENTS.md Section L).*

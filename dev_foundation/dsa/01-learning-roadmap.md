# DSA Learning Roadmap

**Created Date:** 2026-09-28
**Last Updated:** 2026-09-28
**Tags:** `#DSA`, `#Algorithms`, `#Roadmap`, `#Planning`
**Status:** Draft
**Difficulty:** Beginner → Advanced
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- DSA is learned by **writing code and being wrong**, not by watching lectures. Every note here is a *preparation*; the real work happens in the code repo.
- The order is **linear structure → search/sort → trees → graphs → DP**, with the reasoning that each phase adds one new *kind* of problem, not just new syntax.
- The CLI skills from [`../linux/`](../linux/README.md) are not separate from DSA — running, timing, and reading errors *is* half of algorithmic practice.
- Progress is measured by **problems solved without hints**, not by topics read.

---

## 🧠 Detailed Notes

### 1. Why DSA is the backbone of this learning plan

Three reasons, in order of importance:

1. **It trains the thing that transfers to everything else.** Choosing an algorithm is the same act as choosing a database index, a container orchestration strategy, or a model architecture: *given a problem shape and a constraint, what is the cheapest sufficient mechanism?*
2. **It is the most honest feedback loop available.** A test either passes or fails. Compare that with "I read 20 pages about Transformer" — no signal at all.
3. **It is portable.** Languages change; algorithmic thinking doesn't. That matters for an AI Engineer career where the tooling churns every 18 months.

**But:** DSA is *not* the goal. It is the training ground. The output I actually want is the *transfer* — the ability to analyse an unfamiliar system, decompose it, and reason about cost.

### 2. Principles for this track

| Principle | What it means in practice |
|-----------|---------------------------|
| **Derive, don't memorise** | Before looking up a solution, write down: brute force → what makes it slow → what property is wasted → improved version. The *derivation* is the note worth keeping. |
| **Always state complexity first** | Write the $O$ and $O(n)$ space **before** running anything. If you cannot, you have not solved it yet. |
| **Implement from scratch at least once** | Every core data structure (hash map, heap, graph, trie) implemented by hand once. It removes a whole class of bugs later. |
| **Two languages, one logic** | Primary **C++** (closest to how judges and interviews judge algorithmic intent), secondary **Python** (fast to prototype, and the language of the AI stack). Same idea, two notations — the notation stops being the bottleneck. |
| **Errors are the content** | Keep an error journal: what went wrong, why, and the smallest test that reproduces it. See `journal/` in the code repo. |
| **Time-boxed, not endless** | A session is 60–90 minutes with a fixed problem count. Burnout produces notes, not skill. |

### 3. Phased roadmap

Each phase lists: **what to learn → what "done" looks like → where the notes go.**

---

#### Phase 0 — Tooling (parallel with Linux, ~1 week)
- **Learn:** compiling & running (`g++`, `python`), reading errors, timing (`time ./a.out`), basic Git per-solution commits.
- **Done when:** you can compile a solution, get a wrong answer, print debug output, find the bug, and commit the fix — all without leaving the terminal.
- **Where:** [`../linux/`](../linux/README.md) notes 01–03; code repo `solutions/01-fundamentals/`.

#### Phase 1 — Fundamentals (~2–3 weeks)
- **Learn:** arrays, strings, two pointers, sliding window, sorting, binary search, hash maps/sets, basic recursion.
- **Done when:** you can pick the right one of these for a problem in under 2 minutes, and implement each from scratch.
- **Exit test:** solve 15 easy problems with no hints.

#### Phase 2 — Linear structures & queues (~2 weeks)
- **Learn:** stack, queue, deque, monotonic stack/queue, linked list, prefix sums.
- **Done when:** you recognise "the answer needs a running maximum/minimum" and know monotonic structures are the answer.
- **Exit test:** monotonic-stack problems without hints.

#### Phase 3 — Trees & traversal (~3–4 weeks)
- **Learn:** DFS/BFS, recursion on trees, BST operations, traversal orders, heaps/priority queues, tries.
- **Done when:** you can pick the right traversal (pre/in/post/level-order) and justify it in one sentence.
- **Exit test:** tree recursion problems (e.g. max depth, path sum, LCA) without hints.

#### Phase 4 — Graphs (~4 weeks)
- **Learn:** adjacency list vs matrix, BFS/DFS on graphs, topological sort, union-find, shortest path (Dijkstra), bipartite check, cycle detection.
- **Done when:** you can decide *before coding* whether a graph problem is a traversal, a topological problem, or a shortest-path problem.
- **Exit test:** clone-a-graph, course schedule, number of islands without hints.

#### Phase 5 — Dynamic programming (~4–6 weeks)
- **Learn:** 1D DP, 2D DP, knapsack, LIS/LCS, interval DP, bitmask intro, memoisation vs tabulation.
- **Done when:** you can identify the **state definition** in a sentence before writing the transition.
- **Exit test:** classic 1D and 2D DP problems; be able to explain the state to a non-programmer.

#### Phase 6 — Advanced / optional
- **Learn:** greedy + proof, backtracking & pruning, bit manipulation, number theory, string algorithms (KMP, Z), advanced graphs (MST, SCC), segment trees.
- **Done when:** you know which of these are "contest-only" and which are genuinely used in engineering (segment trees, MST, bit tricks are; most DP patterns are interview-specific).

### 4. How this connects to the rest of `dev_foundation/`

| DSA concept | Where else it shows up |
|-------------|------------------------|
| Hash map (average $O(1)$) | Database hash indexes, JSON parsing, counting in log analysis |
| Heap / priority queue | Joblink notifications ordering, task schedulers in `software_architecture/01` |
| Graph traversal | Social-network features in Joblink, dependency graphs, pipeline DAGs |
| Topological sort | Build systems, CI pipeline ordering, DAG schedulers (Airflow-style) |
| B-tree / balanced trees | Every database index — see [`../databases/`](../databases/README.md) |
| Complexity $O(n \log n)$ | Sorting cost in data pipelines, MLOps feature engineering |
| Bit manipulation | Memory/network protocols, hash functions, model quantisation |

### 5. Progress tracker

> Update this table weekly. Source of truth for *what's done* is the code repo; this table is the diary-side view.

| Phase | Status | Problems solved (no hints) | Notes in this repo | Code repo folder |
|-------|--------|---------------------------|--------------------|------------------|
| 0 — Tooling | ⬜ Not started | 0 | 0 | `solutions/01-fundamentals/` |
| 1 — Fundamentals | ⬜ Not started | 0 | 0 | `solutions/01-fundamentals/` |
| 2 — Linear structures | ⬜ Not started | 0 | 0 | `solutions/02-linear-structures/` |
| 3 — Trees | ⬜ Not started | 0 | 0 | `solutions/03-trees/` |
| 4 — Graphs | ⬜ Not started | 0 | 0 | `solutions/04-graphs/` |
| 5 — Dynamic programming | ⬜ Not started | 0 | 0 | `solutions/07-dynamic-programming/` |
| 6 — Advanced | ⬜ Not started | 0 | 0 | `solutions/08-greedy/`, `09-backtracking/`, `10-math-number-theory/` |

**Legend:** ⬜ Not started · 🟡 In progress · 🟢 Done

**Weekly target:** 5–8 problems (2 sessions × 3–4 problems), of which at least 2 must be solved without hints. Consistency beats intensity.

### 6. Platform mix

| Platform | Use for | Why |
|----------|---------|-----|
| **LeetCode** | Interview-shaped, tagged practice | Best for phase-specific drilling by topic |
| **Codeforces** | Contest speed + rating under time pressure | The honest test of real fluency |
| **VNOI / CSES / Kattis** | Classic algorithm suites | Structured, curated, no account needed |
| **PTIT** | Course-aligned exercises | Aligns with university assignments (SE courses) |
| **Project Euler** | Math-heavy problems | Only if the math interest appears |

---

## 📝 Research Journey
- **Why:** I want an AI Engineer who can reason about cost and structure, not just call APIs. Every AI system I admire (a vector index, a serving layer, a training loop) is an algorithmic decision wearing an ML hat.
- **Struggle:** The temptation to *read about* algorithms instead of *implementing* them. Reading feels productive; implementing is slow and produces visible failure. Also: bouncing between too many platforms without depth in any of them.
- **Aha moment:** Writing the brute force first, *timing it* with `time ./a.out`, and then deriving the improvement myself. The timing output turns "this feels slow" into a measured fact, and the improvement stops being a magic trick.
- **Career link:** For an AI Engineer, this track is the *engineering floor*: data pipeline cost, retrieval search complexity, model training memory layout, and inference batching are all complexity decisions. Interview conversations about system design become much sharper when $O$ intuition is automatic.

---

## 🔗 Related Topics
- [`README.md`](README.md) — the anchor note for this folder
- [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md) — CLI skills used in every DSA session
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — why `chmod +x` matters for your solution scripts
- [`../../git_github/03-core-daily-workflow.md`](../../git_github/03-core-daily-workflow.md) — commit discipline per solution
- [`../databases/README.md`](../databases/README.md) — B-trees and hash indexes are Phase 6 structures in production
- [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md) — scaling curves, the business twin of complexity
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there; this diary holds roadmap and reflections
- [SUGGESTED] `02-complexity-analysis-toolkit.md` — amortized analysis, when $O$ lies, and how to measure for real
- [SUGGESTED] `03-data-structure-selection-guide.md` — map problem shape → structure, so solutions are derived not recalled

## 🤔 Open Questions
- [ ] C++ standard target: C++17 or C++20? (affects which STL tools are fair game on judges)
- [ ] Should Python be the primary language given the AI/ML direction, with C++ second?
- [ ] Which platform becomes the *daily driver* — one platform used daily beats five used occasionally?
- [ ] How to keep DSA notes from sprawling: one note per topic here, or one note per month?
- [ ] Target: what does "done" mean — a rating on Codeforces, a number of problems, or interview readiness?

---
*Roadmaps are useful only if revisited. Update the tracker, then let the code tell the truth.*

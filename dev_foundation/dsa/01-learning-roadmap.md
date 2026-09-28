# DSA Learning Roadmap

**Created Date:** 2026-09-28
**Last Updated:** 2026-09-28
**Tags:** `#DSA`, `#Algorithms`, `#Roadmap`, `#Java`, `#C++`, `#Planning`
**Status:** Draft
**Difficulty:** Beginner → Advanced
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- DSA is learned by **writing code and being wrong**, not by watching lectures. Every note here is a *preparation*; the real work happens in the code repo.
- The order is **linear structure → search/sort → trees → graphs → DP**, with the reasoning that each phase adds one new *kind* of problem, not just new syntax.
- The CLI skills from [`../linux/`](../linux/README.md) are not separate from DSA — running, timing, and reading errors *is* half of algorithmic practice.
- Progress is measured by **problems solved without hints**, not by topics read.
- **Two languages, two purposes: Java for the course, C++ for interviews.** They are not interchangeable exercises of the same skill — each teaches a different instinct.

---

## 🧠 Detailed Notes

### 1. Why DSA is the backbone of this learning plan

Three reasons, in order of importance:

1. **It trains the thing that transfers to everything else.** Choosing an algorithm is the same act as choosing a database index, a container orchestration strategy, or a model architecture: *given a problem shape and a constraint, what is the cheapest sufficient mechanism?*
2. **It is the most honest feedback loop available.** A test either passes or fails. Compare that with "I read 20 pages about Transformer" — no signal at all.
3. **It is portable.** Languages change; algorithmic thinking doesn't. That matters for an AI Engineer career where the tooling churns every 18 months.

**But:** DSA is *not* the goal. It is the training ground. The output I actually want is the *transfer* — the ability to analyse an unfamiliar system, decompose it, and reason about cost.

### 2. The language policy: two languages, two purposes

> This section is the plan's load-bearing decision. It is mirrored in `AGENTS.md` Section N.

| Language | Role | Why | Detail note |
|----------|------|-----|--------------|
| **Java** | **Course language — do the assignments in Java** | The DSA course materials are Java, so the exam/exam-style assessments are Java. Practising in a different language than the one you are graded in adds a pointless translation step under time pressure. | [02-java-for-dsa.md](02-java-for-dsa.md) |
| **C++** | **Interview & algorithmic fluency** | Closest to how algorithmic intent is judged in interviews and competitive programming: value semantics, explicit memory, tight constant factors, STL with real priority queues and fast I/O. | [03-cpp-for-dsa.md](03-cpp-for-dsa.md) |

**Deliberately *not* DSA languages here:**

| Language | Where it belongs instead | Why excluded from this track |
|----------|---------------------------|------------------------------|
| **JavaScript** | [`../web_basics/`](../web_basics/README.md) — the **web track** | No built-in `PriorityQueue`/heap, hidden-class costs on dynamic objects, shallow call stack. It is a web language, not a DSA language. |
| **Python** | [`../../ai_ml/`](../../ai_ml/README.md), `Snake_ANN` | Excellent for prototyping and for ML work, but the course is graded in Java and interviews in C++. A third DSA language would split attention without adding a graded skill. |

**How the two DSA languages are used together — the rule:**

```
1. Attempt the problem in JAVA first (the graded language).
2. If the idea is correct and you want to sharpen the algorithm itself,
   PORT it to C++ (or write it from scratch in C++ without looking).
3. In approach.md, write the "Language Notes" section: what each language
   forced you to do differently, and what each one made easy.
```

**Why port instead of doing everything twice:** the *algorithm* is the skill; the language is the medium. Porting once forces you to notice which parts of your solution are language-specific and which are the actual idea. That noticing is the transferable part. Doing every problem twice from scratch is just double the workload for the same insight.

**One thing to watch:** knowing an algorithm in one language can hide a bug in the other. That is exactly why step 3 (Language Notes) is mandatory in the write-up template.

### 3. Principles for this track

| Principle | What it means in practice |
|-----------|---------------------------|
| **Derive, don't memorise** | Before looking up a solution, write down: brute force → what makes it slow → what property is wasted → improved version. The *derivation* is the note worth keeping. |
| **Always state complexity first** | Write the $O$ and $O(n)$ space **before** running anything. If you cannot, you have not solved it yet. |
| **Implement from scratch at least once** | Every core data structure (hash map, heap, graph, trie) implemented by hand once, in **Java**. It removes a whole class of bugs later. |
| **Two languages, one logic** | Java and C++ are not two skills — they are two renderings of the same idea. If the C++ version is a line-by-line transliteration with a different syntax, the exercise was wasted. Port the *idea*, then simplify it with the target language's strengths. |
| **Errors are the content** | Keep an error journal: what went wrong, why, and the smallest test that reproduces it. See `journal/` in the code repo. |
| **Time-boxed, not endless** | A session is 60–90 minutes with a fixed problem count. Burnout produces notes, not skill. |

### 4. Phased roadmap

Each phase lists: **what to learn → what "done" looks like → where the notes go.**

---

#### Phase 0 — Tooling (parallel with Linux, ~1 week)
- **Java:** `javac`/`java`, `jshell` for quick experiments, reading stack traces, `BufferedReader`/`FastScanner` for input.
- **C++:** `g++ -std=c++17 -O2`, `time ./a.out`, reading compiler errors, `ios::sync_with_stdio(false)` for input.
- **Both:** compile → run → print debug output → find the bug → commit the fix, without leaving the terminal.
- **Where:** [`../linux/`](../linux/README.md) notes 01–03; [02-java-for-dsa.md](02-java-for-dsa.md), [03-cpp-for-dsa.md](03-cpp-for-dsa.md); code repo `solutions/01-fundamentals/`.

#### Phase 1 — Fundamentals (~2–3 weeks)
- **Learn:** arrays, strings, two pointers, sliding window, sorting, binary search, hash maps/sets, basic recursion.
- **Java specifics:** `int[]` vs `ArrayList` vs `StringBuilder`; `HashMap<Integer,Integer>` boxing cost; `Arrays.sort` vs `Integer[]` boxing; `Scanner` is too slow — use a hand-written `FastScanner`.
- **C++ specifics:** `vector<int>` vs `vector<long long>`; `sort` on a `vector` vs `sort(begin, end)` on a raw array; `unordered_map` (expected $O(1)$, worst case $O(n)$ — adversarial keys are a real contest trap).
- **Exit test:** solve 15 easy problems in **Java**, no hints.

#### Phase 2 — Linear structures (~2 weeks)
- **Learn:** stack, queue, deque, monotonic stack/queue, linked list, prefix sums.
- **Java specifics:** `ArrayDeque` for stacks/queues (not `Stack` — legacy), `LinkedList` is almost never the right answer; Java has **no built-in monotonic structure** → write it.
- **C++ specifics:** `std::stack`, `std::queue`, `std::deque` are thin wrappers over `deque`; `list` has terrible cache behaviour.
- **Exit test:** monotonic-stack problems in Java, no hints.

#### Phase 3 — Trees & traversal (~3–4 weeks)
- **Learn:** DFS/BFS, recursion on trees, BST operations, traversal orders, heaps/priority queues, tries.
- **Java specifics:** `PriorityQueue` **exists** and is a real binary heap, but it is a *max*-heap by default and has no `decrease-key` → Dijkstra needs an index map; the call stack is the natural recursion limit (deep trees → `StackOverflowError`, not a catchable bug).
- **C++ specifics:** `std::priority_queue<T, vector<T>, greater<T>>` for a min-heap; iterative traversal to avoid stack overflow on skewed trees.
- **Exit test:** tree recursion problems (max depth, path sum, LCA) in Java, no hints.

#### Phase 4 — Graphs (~4 weeks)
- **Learn:** adjacency list vs matrix, BFS/DFS on graphs, topological sort, union-find, shortest path (Dijkstra), bipartite check, cycle detection.
- **Java specifics:** `ArrayList<Integer>[] graph` (generic array creation needs a cast); for large graphs prefer a flat `int[]` adjacency with head/next arrays — same trick as competitive C++; `PriorityQueue` lazy-deletion Dijkstra.
- **C++ specifics:** `vector<vector<int>>`; the flat adjacency list (`head[]`/`to[]`/`nxt[]`) is the performance answer when the judge is strict.
- **Exit test:** clone-a-graph, course schedule, number of islands in Java, no hints.

#### Phase 5 — Dynamic programming (~4–6 weeks)
- **Learn:** 1D DP, 2D DP, knapsack, LIS/LCS, interval DP, bitmask intro, memoisation vs tabulation.
- **Java specifics:** 2D `int[][]` = rows are separate objects (pointer chasing + cache misses); `long[][]` for values that overflow `int`; recursion + memo hits `StackOverflowError` on large states → bottom-up tabulation is often *necessary*, not just faster.
- **C++ specifics:** contiguous `vector<vector<int>>` has the same row-pointer issue but a much smaller constant; use a flat 1D index (`dp[i*W + j]`) when it matters.
- **Exit test:** classic 1D and 2D DP in Java; be able to explain the state to a non-programmer.

#### Phase 6 — Advanced / optional
- **Learn:** greedy + proof, backtracking & pruning, bit manipulation, number theory, string algorithms (KMP, Z), advanced graphs (MST, SCC), segment trees.
- **Java specifics:** `BigInteger` for problems that exceed 64 bits (rare on exams, common in number theory); bit ops on `int` vs `long`; `String.hashCode()` and why it is *not* a secure hash.
- **C++ specifics:** `__int128` for intermediate products before modulo; `unsigned long long` when the answer may be negative-shifted.
- **Done when:** you know which of these are "contest-only" and which are genuinely used in engineering (segment trees, MST, bit tricks are; most DP patterns are interview-specific).

### 5. How this connects to the rest of `dev_foundation/`

| DSA concept | Where else it shows up |
|-------------|------------------------|
| Hash map (average $O(1)$) | Database hash indexes, JSON parsing, counting in log analysis |
| Heap / priority queue | Joblink notifications ordering, task schedulers in `software_architecture/01` |
| Graph traversal | Social-network features in Joblink, dependency graphs, pipeline DAGs |
| Topological sort | Build systems, CI pipeline ordering, DAG schedulers (Airflow-style) |
| B-tree / balanced trees | Every database index — see [`../databases/`](../databases/README.md) |
| Complexity $O(n \log n)$ | Sorting cost in data pipelines, MLOps feature engineering |
| Bit manipulation | Memory/network protocols, hash functions, model quantisation |
| **Java ↔ C++ contrast** | Backend trade-offs: JVM startup/GCC vs native performance — see [`../../software_architecture/`](../../software_architecture/README.md) |

### 6. Progress tracker

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

**Per-problem language target:** every problem is **attempted in Java**. Port to C++ only when the problem is in a topic where C++'s tooling is a genuine advantage (heaps in Phase 3–4, number theory in Phase 6) or when preparing for an interview.

### 7. Platform mix

| Platform | Use for | Why |
|----------|---------|-----|
| **Course platform / exercises** | The graded work | The language and difficulty match what is actually examined |
| **LeetCode** | Interview-shaped, tagged practice | Best for phase-specific drilling; supports Java and C++ |
| **Codeforces** | Contest speed under time pressure | The honest fluency test; C++ is the practical language here |
| **VNOI / CSES / Kattis** | Classic algorithm suites | Structured, curated, often no account needed |
| **PTIT** | Course-aligned exercises | Aligns with university assignments (SE courses) |
| **Project Euler** | Math-heavy problems | Only if the math interest appears |

---

## 📝 Research Journey
- **Why:** I want an AI Engineer who can reason about cost and structure, not just call APIs. Every AI system I admire (a vector index, a serving layer, a training loop) is an algorithmic decision wearing an ML hat.
- **Struggle:** The temptation to *read about* algorithms instead of *implementing* them. Reading feels productive; implementing is slow and produces visible failure. Also: the language question — I was going to learn three or four languages for DSA, which looks productive and is actually the most expensive form of procrastination.
- **Aha moment:** Writing the brute force first, *timing it* with `time ./a.out`, and then deriving the improvement myself. The timing output turns "this feels slow" into a measured fact, and the improvement stops being a magic trick.
- **Career link:** For an AI Engineer, this track is the *engineering floor*: data pipeline cost, retrieval search complexity, model training memory layout, and inference batching are all complexity decisions. Interview conversations about system design become much sharper when $O$ intuition is automatic.

---

## 🔗 Related Topics
- [README.md](README.md) — the anchor note for this folder
- [02-java-for-dsa.md](02-java-for-dsa.md) — the course language: collections, generics, overflow, no built-in monotonic structures
- [03-cpp-for-dsa.md](03-cpp-for-dsa.md) — the interview language: STL, templates, fast I/O, flat adjacency
- [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md) — CLI skills used in every DSA session
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — why `chmod +x` matters for your compile/run scripts
- [`../web_basics/README.md`](../web_basics/README.md) — the *other* track: JavaScript for the web
- [`../databases/README.md`](../databases/README.md) — B-trees and hash indexes are Phase 6 structures in production
- [`../../git_github/03-core-daily-workflow.md`](../../git_github/03-core-daily-workflow.md) — commit discipline per solution
- [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md) — scaling curves, the business twin of complexity
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there; this diary holds roadmap and reflections
- [SUGGESTED] `04-complexity-analysis-toolkit.md` — amortized analysis, when $O$ lies, and how to measure for real
- [SUGGESTED] `05-data-structure-selection-guide.md` — map problem shape → structure, so solutions are derived not recalled

## 🤔 Open Questions
- [ ] Java target version for the course: Java 8 (`var`, streams), Java 11, or Java 17+? Affects which APIs are fair game.
- [ ] C++ standard target: C++17 or C++20?
- [ ] Which platform becomes the *daily driver* — one platform used daily beats five used occasionally?
- [ ] How many problems should be ported to C++ before the C++ track stops paying off? (suggested rule of thumb: 1 in 5, weighted toward heaps/number theory)
- [ ] Do we keep a Python note at all, for the `ai_ml/` link, or does that live entirely in `ai_ml/`?
- [ ] Target: what does "done" mean — a course grade, a Codeforces rating, a problem count, or interview readiness?

---
*Roadmaps are useful only if revisited. Update the tracker, then let the code tell the truth.*

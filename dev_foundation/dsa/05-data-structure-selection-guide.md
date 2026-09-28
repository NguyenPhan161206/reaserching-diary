# Data Structure Selection Guide — Choosing by Problem Shape

**Date:** 2026-09-28
**Status:** Draft
**Difficulty:** Intermediate → Advanced
**Template:** A (Tech Note) — adapted
**Tags:** `#DSA`, `#DataStructures`, `#DecisionMaking`, `#ProblemDecomposition`, `#FenwickTree`, `#SegmentTree`, `#MonotonicStack`
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- **Never start from a structure name. Start from an operations inventory.** Before writing code, write down: which operations, how often, and on what data. The structure is an *output* of that table, not an input to your thinking. Most "I don't know how to solve this" moments are an un-written inventory.
- **The hottest operation picks the structure.** If membership is queried $n$ times while insertion happens $n$ times, you need a hash set. If you instead need the *smallest* remaining item $n$ times, you need a heap. Same data, different question, different structure.
- **Order is a feature, not decoration.** A hash map cannot answer "give me the k-th smallest", "what is the next element after $x$", or "all values in $[l, r]$". The moment a problem needs order, an ordered structure (or sort-then-index) is forced.
- **Algebra decides which aggregate structure is legal.** Prefix sums and Fenwick trees need an **invertible** operation; segment trees need only **associativity**; sparse tables need **idempotence**. Knowing *which property your operation has* tells you which structures are even possible — before you write a line.
- **Offline beats online more often than you think.** If all queries are given up front, sorting by the query parameter and sweeping converts a whole class of "hard" dynamic problems into two pointers + one data structure.
- **Monotonicity is a superpower.** Problems that look like they need a balanced tree or a segment tree ("next greater", "largest rectangle", "sliding window max") collapse to $O(n)$ with a stack or deque, because the answer sequence is monotone.
- **Java and C++ give you almost none of the advanced structures.** Fenwick, segment tree, DSU, trie, monotonic deque — all written from scratch, in both languages. That is not a gap to apologise for; writing them once is how they become yours.

---

## 🧠 Detailed Notes

### 1. The framework: the operations inventory

This is the transferable skill. Everything else in this note is a lookup table for it.

**Step 1 — list the operations, with their frequency, before writing any code.**

| Operation | Frequency | Notes / constraints |
|-----------|-----------|----------------------|
| `contains(x)` | $n$ times | Is the key an integer? Bounded? |
| `get(x)` | $n$ times | Value type? Default when absent? |
| `insert(x)` | $n$ times | Duplicates allowed? Need order? |
| `remove(x)` | 0 | If never: you may not need a deletable structure |
| `min()` / `max()` | $n$ times | Once per element, or once per query? |
| `k-th smallest` | $q$ times | With updates? Or a static array? |
| `range [l, r]` | $q$ times | Is the operation associative / invertible / idempotent? |
| `prefix` | $q$ times | Sum, XOR, min, or max? |
| in-order / next / previous | ? | Does order matter? |
| `size()` | ? | Frequently — the cheapest way to turn a question into a different one |

**Step 2 — mark the hot operation.** One operation dominates. Build the structure for it and make the rest "good enough". Optimising the wrong operation is the most common waste in DSA and the most common waste in production schema design.

**Step 3 — write the structural requirement each candidate implies.** Not the complexity — the *capability*.

**Step 4 — check availability in your language** (see §5).

**Worked example — "count pairs $(i, j)$, $i < j$, with $a_i < a_j$"** (the classic merge-sort inversion count, with `$<$` instead of `$>`; LeetCode 493 *Reverse Pairs* is the same divide-and-conquer with the scaled predicate $a_i > 2a_j$):

| Operation | Frequency | Implication |
|-----------|-----------|-------------|
| scan left to right | $n$ | Outer loop is free |
| "how many earlier values are $< a_j$?" | $n$ times | The hot operation |
| values can be huge (10⁹) or negative | — | Cannot use a raw array → compress |
| duplicates matter (strictly less) | — | Query "count of $\le v-1$" |
| static, no updates | — | A BIT is allowed; a segment tree is overkill |

Derivation: scan left to right, and for each $a_j$ ask "how many of the values I have already seen are smaller than $a_j$?". Values are unbounded, so **compress** them to ranks $1..m$, keep a Fenwick tree of counts, and query the prefix up to $\text{rank}(a_j) - 1$. Total $\Theta(n \log n)$.

Notice what did **not** happen: no mention of a tree until the inventory demanded one. The $O(n^2)$ version is the same problem with the hot operation performed by a linear scan.

### 2. The decision matrix: problem shape → structure

| Problem shape | Canonical structure | Cost | Why it fits |
|---------------|--------------------|------|-------------|
| membership / counting, unordered | hash set / map | $O(1)$ avg | No order needed; lookups dominate |
| k-th smallest, static array | quickselect | $\Theta(n)$ avg | One pass, no sort, mutates the array |
| k-th smallest, stream or many queries | min/max-heap of size $k$ | $O(n \log k)$ | Keep only the $k$ best so far |
| k-th smallest with bounded values | counting / bucket | $O(n + k)$ | Values are already a small domain |
| k-th smallest, dynamic set | order-statistics tree, or BIT over compressed ranks | $O(\log n)$ | Needs order *and* selection |
| sliding window max/min (live) | monotonic deque | $O(n)$ amort | Monotone frontier, each element enters/leaves once |
| sliding window max/min (static) | sparse table or segment tree | $O(1)$ / $O(\log n)$ per query | No updates → idempotent ops |
| next greater / previous smaller | monotonic stack | $O(n)$ amort | Same frontier idea, vertical |
| range sum, no updates | prefix sums | $O(n)$ build, $O(1)$ query | Needs an **invertible** op |
| point update + range sum | Fenwick tree (BIT) | $O(\log n)$ | Invertible prefix aggregates |
| range update + point query | BIT over a difference array | $O(\log n)$ | Flip the roles of update and query |
| range update + range query | two BITs, or lazy segment tree | $O(\log n)$ | — |
| arbitrary (non-commutative) range op | segment tree | $O(\log n)$ | Associativity is enough |
| static, idempotent range op (min/max/gcd) | sparse table | $O(n \log n)$ build, $O(1)$ | Idempotence allows overlap |
| order matters, dynamic keys | balanced BST / `TreeMap` | $O(\log n)$ | Ordered traversal, `floor`/`ceiling` |
| always pop the extreme, no need for order | heap | $O(\log n)$ | Cheaper than a BST |
| merge $k$ sorted streams | heap of size $k$ | $O(N \log k)$ | The frontier is always $k$ candidates |
| dynamic connectivity | DSU (+ union by rank, path compression) | $O(\alpha(n))$ | Connectivity is a partition, not an order |
| offline connectivity with time edges | DSU + **rollback** | $O(q \log q)$ | Reverse time, undo merges |
| shortest path, non-negative | Dijkstra + heap | $O((V+E) \log V)$ | Greedy is valid because distances only grow |
| shortest path, negative edges | Bellman–Ford / SPFA | $O(VE)$ | Greedy is invalid; relax until stable |
| DAG paths / scheduling | topological sort + DP | $O(V + E)$ | No cycles → order exists |
| all-pairs, small $V$ | Floyd–Warshall | $O(V^3)$ | $V \le \sim 400$ in practice |
| string prefix queries, many keys | trie | $O(L)$ per op | Shared prefixes are stored once |
| single pattern in a long text | KMP / Z-function | $O(n + m)$, no collisions | Deterministic |
| many pattern checks / fingerprints | rolling hash (double) | $O(n + m)$, probabilistic | Monte Carlo; verify if correctness is critical |
| LRU / LFU cache | hash map + doubly linked list (+ buckets) | $O(1)$ | $O(1)$ needs *both* |
| interval overlap / merge | sort + sweep line | $O(n \log n)$ | Order by start, then scan |
| 2-D range sums | 2-D prefix sums, 2-D BIT, or sweep line | $O(1)$ / $O(\log^2 n)$ | Depends on updates |

### 3. The algebra rule: which structure is even legal

This is the part tutorials usually skip, and it is why choosing "correctly" feels like guesswork. The aggregate structures differ in *exactly one* requirement:

| Structure | Requires of the aggregate | If your op is | Then |
|-----------|--------------------------|---------------|------|
| prefix sums | **invertible**: given the total, recover the removed part (subtraction) | max, min, gcd, OR, AND, median | ❌ impossible — you cannot "un-add" a maximum |
| Fenwick tree | **invertible** (same reason, over prefixes) | sum, XOR, difference, count | ✅ |
| segment tree | **associative** + identity only | sum, max, min, product, matrix multiply, max-subarray | ✅ order-sensitive aggregates allowed |
| sparse table | **idempotent** ($f(f(x, y), y) = f(x, y)$) | min, max, gcd, bitwise OR | ✅ overlapping ranges are safe; sum is not |
| hash map | no property, but **no order** | anything, unordered | ⚠️ you must sort afterwards for ordered output |
| ordered tree | total order on keys | predecessor, successor, k-th by rank | ✅ |
| heap | total order + only "extreme" access | repeatedly pop the max/min | ✅ |
| monotonic structure | a **total order that the operation preserves** | next greater, next smaller, largest rectangle | ✅ if monotone, else ❌ |

**Worked example — the same question, two answers.** "Range maximum over $[l, r]$, no updates":

- prefix sums → **impossible** (no inverse for max)
- segment tree → possible, $O(\log n)$
- sparse table → possible, $O(1)$ per query, $O(n \log n)$ memory
- monotonic stack → possible *offline*: answer queries sorted by right endpoint, maintaining a decreasing stack, popping when the new element dominates. $O(n + q)$ total.

**Worked example — why Fenwick cannot do "range min update" naively.** Fenwick stores prefix aggregates; a range update changes two points of the difference array, and querying a point needs a *prefix sum* of the difference array. It works. But range *min* update with point query requires the difference of minima, which is meaningless — min is not invertible. **The same-looking problem, one operation apart, is solvable and unsolvable with the same data structure.** That is why the inventory asks for the operation, not the shape.

### 4. Monotonicity: the $O(n)$ escape hatch

A surprising number of "hard" problems are a balanced structure used for a task a stack or deque does in one pass. The mechanism: maintain a frontier where dominated candidates are **discarded permanently**.

```java
// Next greater element to the right, for every index — Θ(n), not Θ(n log n)
int[] nextGreater(int[] a) {
    int n = a.length, ans[] = new int[n];
    Arrays.fill(ans, -1);
    Deque<Integer> stack = new ArrayDeque<>();   // values are STRICTLY decreasing
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && stack.peek() < a[i]) {
            ans[stack.pop()] = a[i];             // a[i] dominates the popped value
        }
        stack.push(a[i]);
    }
    return ans;
}
```

The reasoning, which is the actual skill: when index $i$ arrives, every smaller value waiting at the top of the stack can never be a "next greater" candidate again, because $a_i$ is bigger *and* closer. So they are popped and never stored. Each element is pushed once and popped at most once → $\Theta(n)$ total, no logarithm, no tree.

The same argument, rotated, gives: sliding window maximum (deque instead of stack), largest rectangle in histogram, "sort the subarray" (min-heap + indices), k-th smallest of a subarray, and merge-$-k$-sorted-lists. **Whenever you see "for each window/query, the extreme", ask whether a dominated element can be discarded forever.** If yes, it is a monotonic structure.

### 5. What the languages give you (and what you must write)

Per the folder's two-track policy: **Java is the course language, C++ is the port target** ([`02-java-for-dsa.md`](02-java-for-dsa.md), [`03-cpp-for-dsa.md`](03-cpp-for-dsa.md)).

| Structure | Java | C++ | You must write it because |
|-----------|------|-----|--------------------------|
| dynamic array | `ArrayList` | `vector` | provided |
| hash map / set | `HashMap`, `HashSet` | `unordered_map`, `unordered_set` | provided |
| ordered map / set | `TreeMap`, `TreeSet` | `map`, `set` | provided |
| min/max heap | `PriorityQueue` (**max** by default) | `priority_queue<..., greater<T>>` | provided |
| stack / queue / deque | `ArrayDeque` | `stack`, `queue`, `deque` | provided |
| **monotonic stack/deque** | ❌ none | ❌ none | The frontier invariant is the problem |
| **Fenwick tree** | ❌ none | ❌ none | ~20 lines; write it once, reuse forever |
| **segment tree** | ❌ none | ❌ none | ~60 lines; the lazy variant is the real work |
| **disjoint set union** | ❌ none | ❌ none | ~30 lines |
| **trie** | ❌ none | ❌ none | ~40 lines |
| **order-statistics tree** | ❌ none | `__gnu_pbds::tree` (GNU extension, **not** standard) | k-th by rank with updates |
| **coordinate compression** | sort + `Arrays.binarySearch` | sort + `lower_bound` | needed whenever keys are unbounded |
| **KMP prefix function** | ❌ none | ❌ none | `pi` array, ~15 lines |
| big integer | `BigInteger` | `boost::multiprecision` / `__int128` | products of two `long`s |
| `lower_bound` semantics | `Arrays.binarySearch` (index **or** negative insertion point) | `lower_bound` (clean iterator) | Porting trap: Java's return value is not an index |

**The strategic read:** the standard libraries cover the *easy half* of the data-structure world. Everything that requires an invariant — BIT, segment tree, DSU, monotonic structures, tries — is yours to write, in both languages. Budget for that: each one is a weekend, and then it is reusable for the rest of the track.

**Porting note (relevant to the two-track rule):** `Arrays.binarySearch`'s "negative insertion point" encoding is a genuine porting trap — in C++ the same call returns an iterator, not an index, and you need `-a.begin()` plus an `insert` check. Write these in the **Language Notes** section of the solution when you hit them.

### 6. The rules that generalise

1. **Inventory first, structure second.** No exceptions. If you cannot fill in the table, you do not yet understand the problem — and the code you write will be a guess.
2. **The hot operation decides; the rest get "good enough".** Optimise the 20% of operations that are 80% of the work.
3. **If order is required, order is required.** `HashMap` + sort at the end is sometimes right (one sort, $O(n \log n)$ total) — but only if you need the order *once*, at the end.
4. **Offline is a gift.** All queries known up front → sort by the query parameter, sweep once. Interval overlap, offline connectivity, static range queries: all become simple.
5. **Repetition is a reason to precompute.** The same expensive query asked $q$ times → prefix sums, sparse table, or memoisation. The build cost must be amortised against $q$.
6. **When you see nothing structural, sort + two pointers + binary search.** It is $O(n \log n)$ and it solves a large fraction of what looks hard.
7. **Check the algebraic property before the structure.** Invertible → prefix/BIT. Associative → segment tree. Idempotent → sparse table. Order-preserving → monotone structure.
8. **Simplest structure that meets the worst case, not the cleverest.** A Fenwick tree that passes beats a segment tree that is elegant and $2\times$ slower and 3× the code.

### 7. Anti-patterns: the wrong turns

| Wrong turn | Symptom | What it really is | Correct move |
|-----------|---------|------------------|--------------|
| Hash map when order is needed | "the output is in the wrong order" / k-th impossible | Missing capability, not a missing speed | `TreeMap`, or hash + one sort at the end |
| Balanced BST when order is never used | Slower than a hash, twice the code | Paying for a feature you do not need | Hash map, or even a sorted array |
| Hash map when keys are small bounded integers | Slow, memory-heavy | Using a general structure for a direct-address problem | Array / bitset of size $k$ |
| `O(n^2)$ scan inside a loop that could use a set | Passes small tests, TLE at $10^5$ | Hot operation not given a structure | `HashSet` ([`02-java-for-dsa.md`](02-java-for-dsa.md)) |
| Sorting inside the loop | $O(n^2 \log n)$ | Precomputation missing | Sort once before the loop |
| Priority queue when a stack/deque would do | Extra $\log n$, more code | Reachable frontier not identified | Stack/deque (§4) |
| Segment tree when a BIT or prefix sum suffices | 3× the code, same complexity | Skipped the invertible-vs-associative check | BIT / prefix sums |
| Fenwick for min/max aggregates | Wrong answers, silently | `min` is not invertible | Segment tree / sparse table |
| Sparse table with updates assumed | Stale results | Sparse table cannot update | Rebuild, or use a segment tree |
| Linked list of objects when indices suffice | Cache misses, ~5–10× memory | Objects where a flat array would do | `int[]` head/next arrays (see C++ note) |
| Recursion where depth is $O(n)$ | `StackOverflowError` / segfault | Mistook tree height for depth | Iterative traversal with an explicit stack |
| DSU for a general dynamic graph with many queries | Wrong (DSU cannot un-merge) | Partition maintenance is one-directional | BFS/DFS per query, or rollback DSU offline |
| Building a structure before reading the constraints | Wrong choice, wasted day | Design before requirements | Read constraints → inventory → structure |
| Assuming the advanced structure is in the library | `NoClassDefFoundError` hunting | Over-trusting the STL/Collections | §5 table: write it yourself |

---

## 📊 Comparison table

| Structure | Search | Insert | Delete | Order | Range queries | Space | Reach for it when |
|-----------|--------|--------|--------|-------|---------------|-------|-------------------|
| unsorted array / `ArrayList` | $O(n)$ | $O(1)$ amort | $O(n)$ (by value) / $O(1)$ (by index) | input order | $O(n)$ scan | $O(n)$ | the default; scan is often fine |
| sorted array | $O(\log n)$ | $O(n)$ shift | $O(n)$ shift | ✅ | `binarySearch` + range scan | $O(n)$ | static data, many "find near $x$" queries |
| dynamic array `vector` | $O(1)$ by index | $O(1)$ amort | $O(1)$ at end | insertion | $O(n)$ | $O(n)$ | the universal workhorse |
| hash map | $O(1)$ avg / $O(n)$ worst | $O(1)$ avg | $O(1)$ avg | ❌ | ❌ | $O(n)$ | membership, counting, lookup |
| balanced BST / `TreeMap` | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | ✅ | $O(\log n + k)$ | $O(n)$ | predecessor/successor, ordered output |
| binary heap | $O(n)$ for arbitrary, $O(1)$ peek | $O(\log n)$ | $O(\log n)$ | partial | ❌ | $O(n)$ | repeatedly need the extreme, not order |
| Fenwick tree | — | $O(\log n)$ point | — | — | $O(\log n)$ prefix | $O(n)$ | invertible aggregates (sum, XOR) |
| segment tree | — | $O(\log n)$ | — | — | $O(\log n)$ | $O(n)$ (4n typical) | arbitrary + lazy range updates |
| sparse table | — | ❌ (static) | ❌ | — | $O(1)$ idempotent | $O(n \log n)$ | static min/max/gcd, many queries |
| monotonic stack/deque | — | $O(1)$ | $O(1)$ | monotone | — | $O(n)$ worst | sliding windows, next greater, histograms |
| DSU | $O(\alpha(n))$ | $O(\alpha(n))$ | ❌ (by union) | ❌ | connectivity only | $O(n)$ | merging, offline connectivity |
| trie | $O(L)$ | $O(L)$ | $O(L)$ | lexicographic | prefix queries | $O(\text{total chars})$ | many shared-prefix string keys |
| string hash | $O(1)$ | $O(1)$ | $O(1)$ | ❌ | ❌ | $O(n)$ | fingerprinting, dedupe, LCP tricks |

---

## 🧪 Practice Tasks (no answers here — attempt first)

> Do the inventory **on paper first** for every task. If you cannot fill the table, stop and re-read the statement.

1. **Inventory drill, 10 problems.** Take ten problems you have solved. For each, write the operations inventory from §1, name the structure the *original* solution used, and mark whether the inventory predicted it. Report the hit rate honestly.
2. **Same problem, five structures.** "Range minimum queries, no updates": solve with (a) linear scan, (b) segment tree, (c) sparse table, (d) offline monotonic stack, (e) BIT — and for BIT, **prove** it cannot work. Then rank them by total cost for $n = 10^5$, $q = 10^5$ and say which you would ship.
3. **The property drill.** For each aggregate, state whether it is invertible / associative / idempotent: sum, product, min, max, XOR, gcd, bitwise OR, concatenation, "max difference". Then, for each, mark which of {prefix sums, BIT, segment tree, sparse table} is *legal*. This table is worth more than the individual structures.
4. **Build the toolkit from scratch (Java, then port to C++).** Implement, in this order: a monotonic deque, a Fenwick tree, a DSU, a trie, a KMP prefix function, and a lazy segment tree. ~20–60 lines each. For each, write a 5-line brute-force checker and fuzz them against random input. Keep them in the code repo as a `structures/` folder with a README.
5. **Derive, do not recall.** Solve "k-th smallest element in an array" four times: quickselect, a heap of size $k$, a counting sort when bounded, and a full sort. Report time, space, mutates-input, and stability for each. Explain which of the four is *only* possible because the array is static.
6. **Offline conversion.** Take five online-sounding problems (interval overlap counts, "elements smaller than the current threshold", dynamic connectivity, subarray distinct counts, k-th smallest with deletions) and convert each into an offline sort-then-sweep problem. State the sort key for each.
7. **Monotonicity hunt.** Find three problems that look like they need a segment tree or a balanced tree, and show the monotonic-stack/deque solution instead. Explain in each case *why the dominated element can be discarded forever*.
8. **The wrong turn diary.** Collect five real wrong turns from your own practice (wrong structure, structure invalidated by a hidden constraint, structure that exists but is unavailable in Java). For each, write the sentence you now say to yourself that would have prevented it. This is a personal checklist, not a note.
9. **Graph structure choice.** For "clone graph", "course schedule", "number of islands", "network delay", "diameter of a tree", "redundant connection", and "cheapest flights with $k$ stops": write the inventory, name the algorithm *and* the structure, and state the cost in $V$ and $E$. The last two deliberately require a different representation (BFS layers, DSU).
10. **Porting task (per the two-track rule).** Take a solution you wrote in Java using a structure you built by hand (BIT or DSU, say), and port it to C++ **from memory**. Write the **Language Notes** section: what you had to remember, what you had to change, and where the port produced a *different* algorithm (e.g. `__gnu_pbds` for order-statistics, `lower_bound` semantics, `sort` UB with a bad comparator).

---

## 🔗 Tech Cross-References (Section K)
- **Databases** — every structure here has a production, persistent, on-disk twin: B-trees (balanced structure + range scans) as indexes, hash indexes, LSM trees (memtable = heap/skiplist, SSTable = sorted structure), and the query planner choosing between them by cost. A `CREATE INDEX` is this note's decision made by the server → [`../databases/README.md`](../databases/README.md)
- **ML / MLOps** — tensors are flat contiguous arrays (the "unsorted array" row, plus a layout argument), embedding lookup is a hash map, queues feed batches through a training loop, and decision trees are literally the recursive structure from §4. Buffer management, batch queues, and sharding are the production versions of the hot-operation rule → [`../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md)
- **Java** — the Collections Framework is the *easy half* of this table; the missing structures in §5 are the reason a Java DSA course expects you to write BIT/DSU/trie by hand → [`02-java-for-dsa.md`](02-java-for-dsa.md)
- **C++** — same structures, plus real order-statistics trees (GNU `pbds`, not standard) and a flat adjacency list for graphs when the judge is strict. This is where the port earns its keep → [`03-cpp-for-dsa.md`](03-cpp-for-dsa.md)
- **Linux** — data structures all the way down: the VFS resolves paths through a **radix tree**, ext4 allocates files through a **B-tree** of directory entries, the page cache is a hash table, and the process list in `/proc` is a linked list. `ls` blocking on a large directory is a data-structure performance question → [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md)
- **Software architecture** — caches (LRU/LFU = §2's hash + list), rate limiters (token bucket = a queue with a time condition), load shedding, and the priority-queue-for-everything trap are all structure decisions made once and lived with for years → [`../../software_architecture/README.md`](../../software_architecture/README.md)
- **Economics & Business** — matching and market-clearing algorithms are priority queues; double-auction price discovery is a sorted structure; auction design *is* mechanism design over data structures → [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md)

## 📝 Research Journey
- **Why:** I recognised the structure name in a *recognised* problem — "monotonic stack for this one" — and had no idea what to do with a problem that merely *resembled* it. Memorised solutions do not generalise; I have failed contests by recognising nothing at all. This note is the attempt to replace recognition with derivation.
- **Struggle:** The algebra rule in §3 came last, after months of choosing a Fenwick tree by vibes. The moment it clicked was realising that a Fenwick tree needs subtraction — a *group* structure — while a segment tree does not. Max has no inverse, therefore no Fenwick tree, therefore every "range minimum with updates" problem must be a segment tree. One sentence of algebra, and it predicted the answer for dozens of problems I had not yet seen.
- **Aha moment:** The operations inventory. Writing the table *before* the code turned three problems I had marked "hard" into recognisable shapes — one turned out to be a hash set, one an offline sort, one a monotonic deque. Nothing about the algorithms changed; only the order in which I thought about them. The second aha: a data structure is a *capability* (order? selection? range? undo?) before it is a cost.
- **Career link:** Every storage and pipeline decision is this note at a larger scale. Choosing a primary key, an index, a partition scheme, a queue type, a cache eviction policy, or a shard key is a structure decision about a problem shape nobody wrote down for you. The person who can write the inventory is the one who can defend the choice in a design review — and the one who recovers when the requirements change.

## 🔗 Related Topics
- [README.md](README.md) — the folder anchor
- [04-complexity-analysis-toolkit.md](04-complexity-analysis-toolkit.md) — the price tag for whatever this guide tells you to buy
- [01-learning-roadmap.md](01-learning-roadmap.md) — which structures belong to which phase
- [02-java-for-dsa.md](02-java-for-dsa.md) — the Collections half of the availability table
- [03-cpp-for-dsa.md](03-cpp-for-dsa.md) — the STL half, plus the structures C++ gives you that Java does not
- [`../databases/README.md`](../databases/README.md) — the production versions with persistence and query planning
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — B-trees and radix trees inside the kernel
- [SUGGESTED] `06-recursion-to-iteration-mental-model.md` — the tree traversal shapes, and both languages' stack limits (already in the folder backlog)
- [SUGGESTED] `06-graphs-representations-and-traversal.md` — adjacency list vs matrix vs flat arrays, as a dedicated decision note once Phase 4 starts
- [SUGGESTED] `bridge-data-structures-and-databases.md` — bridge note: this guide ↔ index selection and query planning, one row of the table at a time

## 🤔 Open Questions
- [ ] Should the code repo get a `structures/` folder with tested implementations (BIT, DSU, trie, lazy segment tree) so no problem ever re-implements them from scratch? That is the highest-leverage hour I will spend this month.
- [ ] Are there structures in the course syllabus that the standard libraries do *not* cover well and that I have not listed here (e.g. skip lists, finger trees, splay trees, treaps)?
- [ ] The course is Java — do I write the advanced structures in Java only, or maintain both implementations from day one? The two-track policy says attempt in Java; does the toolkit library follow the same rule?
- [ ] How much of §3 (the algebra rule) is exam-relevant versus engineering-relevant? It feels like the most valuable content in the note and the least likely to be asked.
- [ ] Should the "wrong turn diary" from Practice Task 8 become a permanent section inside this note, or stay in the code repo as an append-only log?
- [ ] Does PostgreSQL's planner reasoning (cost-based, using real cardinality statistics) deserve a bridge note, or is that covered by `databases/`?

---
*Created: 2026-09-28 — selection note for the DSA track. Implementations, fuzz tests, and solutions live in the code repo.*

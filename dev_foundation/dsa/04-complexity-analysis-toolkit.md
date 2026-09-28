# Complexity Analysis Toolkit — Measuring Before You Optimise

**Date:** 2026-09-28
**Status:** Draft
**Difficulty:** Beginner → Advanced
**Template:** A (Tech Note) — adapted
**Tags:** `#DSA`, `#BigO`, `#Amortized`, `#Recurrence`, `#Benchmarking`, `#LowerBound`
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- **Big-O is a statement about growth, not about speed.** It answers "what happens to the time when I double the input?" It cannot tell you whether your program is fast today. Anything that claims otherwise ("this is $O(n)$ so it is fast") is confusing the two.
- **You need two symbols, not five.** $O(f)$ is an upper bound, $\Theta(f)$ is a tight bound, and in practice you should always be aiming for $\Theta$. The others ($\Omega$, $o$, $\omega$) matter mostly for lower bounds, which is where real insight lives.
- **Every analysis is three questions: what is $n$? what counts as one unit of work? which term dominates?** Most wrong answers are a wrong $n$ or a hidden non-constant inner cost.
- **Nested loops are not automatically $O(n^2)$.** The rule is "add the costs of the statements, then simplify" — a nested loop over a *different* range, an early `break`, or a shrinking window changes the answer.
- **Amortised $\ne$ average $\ne$ expected.** Amortised is a deterministic guarantee over a *sequence* of operations ($O(1)$ amortised append is a promise about $n$ appends in total); average is over inputs; expected is over the algorithm's own randomness. Conflating them is how people justify an algorithm that has one catastrophic input.
- **Space is a cost too, and it is often the binding constraint.** A "clever" $O(n)$ time / $O(n^2)$ space solution fails a memory limit exactly like a slow one fails a time limit.
- **Constants and memory hierarchy dominate below $n \approx 10^5$.** An $O(\log n)$ binary search on a 10-million-element array can lose to an $O(n)$ linear scan, because random access misses cache and a linear scan does not. This is not an exception to the theory; it is the theory plus a machine model the theory omitted.
- **A claim without a measurement is a hypothesis.** Ratio testing ($n \to 2n \to 4n$) tells you the exponent you actually implemented, which is a stronger statement than the class you intended.

---

## 🧠 Detailed Notes

### 1. Why complexity is the first decision, not a footnote

Choosing an algorithm *is* choosing a cost model. The same question — "what is the cheapest sufficient mechanism for this problem shape?" — appears in every layer of this diary:

| Layer | The "complexity" question looks like |
|-------|----------------------------------------|
| DSA | $O(n \log n)$ sort vs $O(n + k)$ counting sort |
| Databases | B-tree index $O(\log n)$ vs hash index $O(1)$ vs a full scan $O(n)$ |
| MLOps | Batch size vs memory: bigger batch, fewer steps, more RAM |
| Caching | Is the hit rate worth the staleness? $O(1)$ read against a re-computation |
| Economics | Economies of scale — the *shape* of the cost curve, not just its position |

**What Big-O is:** a way to compare the growth of two functions while ignoring everything that does not grow — the language, the CPU, the constant factor, the memory layout, the year.

**What Big-O is not:**

- ❌ a runtime prediction (`O(n \log n)$ does not mean "fast")
- ❌ a lower bound (an $O()$ bound is a *permission slip*, not a guarantee — you may be $\Theta(n^2)$ inside an $O(n^2)$ label and not know it)
- ❌ independent of the model (it assumes unit-cost arithmetic and no cache)

The correct mental framing: **Big-O is a filter.** It removes options that cannot work at the target scale, leaving a short list to measure. It is a necessary condition for being competitive, never a sufficient one.

### 2. The symbols — and the two you actually need

| Symbol | Meaning | Formal shape | Use it when |
|--------|---------|--------------|-------------|
| $O(g(n))$ | upper bound | $\limsup \dfrac{f(n)}{g(n)} < \infty$ | you can prove "never worse than" |
| $\Omega(g(n))$ | lower bound | $\liminf \dfrac{f(n)}{g(n)} > 0$ | you can prove "never better than" |
| $\Theta(g(n))$ | tight bound | $f(n) = \Theta(g(n))$ iff both hold | **this is the real answer** |
| $o(g(n))$ | strictly smaller | $\dfrac{f(n)}{g(n)} \to 0$ | rarely needed in practice |
| $\omega(g(n))$ | strictly larger | $\dfrac{f(n)}{g(n)} \to \infty$ | rarely needed in practice |

Two working rules:

1. **Quote $\Theta$, not $O$.** In interviews and in your own notes, "$O(n \log n)$" when you mean "$\Theta(n \log n)$" is a habit that hides sloppiness. If you have not proved the lower bound, write "$O(n \log n)$" *and* say so.
2. **Drop constants only when you have checked them.** $O(3n) = O(n)$ mathematically, but "hash lookup is $O(1)$" hides the difference between a hash lookup and an array index — usually 10–100×. Keep the constant until you have measured it.

The formal definition that makes the rest intuitive:

$$f(n) = O(g(n)) \iff \exists c > 0,\; n_0 : \forall n \ge n_0,\; f(n) \le c\,g(n)$$

Read it as: *after some threshold $n_0$, $f$ never exceeds $c$ times $g$.* The threshold is why Big-O says nothing about small inputs — and small inputs are most of what you will run.

### 3. The toolkit: three questions, then add it up

Before any algebra, answer these. Most wrong analyses are wrong at step 1 or 2, not in the arithmetic.

1. **What is $n$?** Number of array elements? Number of edges? Number of *bits*? For a string, $n$ is the length; for a graph, $n$ is usually $|V| + |E|$; for an integer, the honest input size is its **bit length** $\log_2 v$, which is why a DP over values is $O(v)$ and a DP over bits can be $O(\log v)$.
2. **What counts as $O(1)$?** An `int` comparison, yes. A `long` multiply, yes on a 64-bit machine. A **string comparison is $O(k)$**. An `ArrayList.get(i)` is $O(1)$; an `ArrayList.contains(x)` is $O(n)$ ([`02-java-for-dsa.md`](02-java-for-dsa.md)). Reading a line from stdin is $O(\text{length})$.
3. **Which term dominates?** After summing, look at the largest surviving term. When two terms tie you must say $\Theta(n \log n)$, not "either one".

Then the mechanical method:

```
for every statement: cost(statement) × (number of times it runs) = contribution
sum all contributions → simplify with the dominant-term rule
```

```java
// What is n?           the array length
// What is O(1)?        int comparison, int assignment
// Dominant term?       the inner loop runs O(n) times for each of O(n) outer
int countPairs(int[] a) {
    int count = 0;                       // O(1)
    for (int i = 0; i < a.length; i++)  // O(n)
        for (int j = i + 1; j < a.length; j++)   // O(n) each → O(n²) total
            if (a[i] < a[j]) count++;    // O(1) per iteration
    return count;
}                                        // Θ(n²) time, Θ(1) space
```

**The same code, fixed — and the fix is a data structure, not a micro-optimisation:**

```java
// Count pairs where a[i] < a[j], i < j, in Θ(n log n) — or Θ(n) with bounded values
// The insight: for each a[j], how many EARLIER elements are smaller? Use a sorted
// structure, so the inner "count" becomes a query instead of a scan.
static long countPairsFast(int[] a) {
    Arrays.sort(a);                       // Θ(n log n) once
    // with a BIT: Θ(n log n) total; with a[max] bounded: Θ(n + max) counting sort
    return 0; // (implementation is Practice Task 3)
}
```

**Why the second version is not "the same algorithm faster":** the first one *re-derives* an ordering fact $n$ times; the second derives it once and queries it. That is the real DSA skill — see [`05-data-structure-selection-guide.md`](05-data-structure-selection-guide.md).

### 4. Loops: how to actually read nested code

| Pattern | Cost | Why |
|---------|------|-----|
| Sequential loops | $\sum$ of the two | $O(n) + O(n^2) = O(n^2)$ |
| Nested, same range | product | $\Theta(n^2)$ |
| Nested, **shrinking** inner range | half the product | triangular loop: $\Theta(n^2)$ still, but half the work |
| `for i in 0..n: while j < n and cond(i,j)` | $\Theta(\text{steps})$ | the two-pointer pattern: $j$ never moves back, so $\le 2n$ total |
| `for i in 0..n: for j in 0..n: if cond: break` | $\Theta(\text{steps})$ | often far better than $n^2$; the `break` is the algorithm |
| Multiply loop (`i` over $a$, `j` over $b$) | $\Theta(ab)$ | → $O(n^2)$ only if $b = \Theta(n)$ |
| `for each x: for each y in smaller_set(x)` | $\sum_x \lvert smaller(x) \rvert$ | the sum is the analysis; if it is $O(n)$ the whole thing is |

**The habit that makes this automatic:** count *element visits*, not "loops". A single `for` over $n$ items that does 3 things is still $O(n)$. A "single loop" containing a `contains()` on a list is $O(n^2)$ — and it is the single most common Java DSA bug ([`02-java-for-dsa.md`](02-java-for-dsa.md), §2).

### 5. Recursion: the master theorem and the recursion tree

Write the recurrence first, then look it up. The canonical form is

$$T(n) = a\,T\!\left(\frac{n}{b}\right) + f(n)$$

- $a$ = number of recursive calls per level
- $b$ = how much the size shrinks per call
- $f(n)$ = the work done *outside* the recursion (combining, merging, scanning)

Define $n^{\log_b a}$ — the **critical exponent** — and compare $f(n)$ against it:

| Case | Condition | Result | Example |
|------|-----------|--------|---------|
| 1 | $f(n) = O(n^{\log_b a - \varepsilon})$ | $\Theta(n^{\log_b a})$ | binary search: $T(n)=T(n/2)+\Theta(1)$, $a=1,b=2$ → $\Theta(\log n)$ |
| 2 | $f(n) = \Theta(n^{\log_b a})$ | $\Theta(n^{\log_b a}\log n)$ | merge sort: $T=2T(n/2)+\Theta(n)$ → $\Theta(n \log n)$ |
| 3 | $f(n) = \Omega(n^{\log_b a + \varepsilon})$ | $\Theta(f(n))$ | `copy` then recurse: $T(n)=T(n/2)+\Theta(n)$ → $\Theta(n)$ |

**Case 2 is the interesting one:** the master theorem's "case 1 + case 3 with a log factor" means every level of the tree costs the same as the last, and there are $\log_b n$ levels. That is exactly why merge sort is $\Theta(n \log n)$ and not $\Theta(n^2)$ — the merging work is *reused* across levels, not repeated.

**The master theorem is narrower than people remember.** These two recurrences are *not* exotic — they are ordinary case 1 and case 3 written in a form people mis-parse:

| Recurrence you meet | $a$, $b$, $f(n)$ | Case it actually falls in | Result |
|---|---|---|---|
| $T(n) = 2T(n/2) + \Theta(\log n)$ — "split in half, the combine step costs $\log n$" | $a=2, b=2, f(n)=\log n$ | **1**, because $\log n = O(n^{1-\varepsilon})$ | $\Theta(n)$ |
| $T(n) = 2T(n/3) + \Theta(n)$ | $a=2, b=3, f(n)=n$ | **3**, because $n = \Omega(n^{\log_3 2 + \varepsilon})$ | $\Theta(n)$ |

Two things people get wrong about the theorem itself:

- **$b$ does not have to be a power of 2.** Nothing in the statement requires it. The only requirement is $b > 1$ — the problem has to actually shrink. "My $b$ is 3, the theorem doesn't apply" is not a reason.
- **Case 2 has a log-factor extension.** If $f(n) = \Theta(n^{\log_b a}\,\log^k n)$, the result is $\Theta(n^{\log_b a}\,\log^{k+1} n)$ — one more log. Merge sort is $k=0$; a divide-and-conquer whose combine step costs $n \log n$ is $k=1$.

Recurrences that genuinely need a different move:

| Recurrence | Why the canonical form does not fit | Method |
|------------|----------------------------------------|--------|
| $T(n) = T(n-1) + \Theta(1)$ | There is no $b$ — the subproblem never shrinks, so the recursion tree is a line, not a branching tree | Unroll: it is $\Theta(n)$ — a linear walk |
| $T(n) = T(n/3) + T(2n/3) + \Theta(1)$ | Two different shrink factors, so there is no single $b$ | Akra–Bazzi: $p=1$ (since $\tfrac13+\tfrac23=1$), giving $\Theta(n)$ |
| Quicksort $T(n) = T(k) + T(n-k) + \Theta(n)$ | $k$ is the data, not a constant, so neither $a$ nor $b$ is fixed | **Expected** $\Theta(n \log n)$, worst $\Theta(n^2)$; fix with a random or median-of-three pivot |

**Why $T(n/3) + T(2n/3)$ is $\Theta(n)$ and *not* $\Theta(\log n)$ — the mistake worth memorising.** The reflex is "the problem shrinks geometrically, so there are $\log n$ levels, and each level is constant, so $\Theta(\log n)$". That reflex is valid for $T(n)=T(n-1)+\Theta(1)$ (one path) and for $T(n)=T(n/2)+\Theta(1)$ (one call per node, so one node per level). It breaks here because there are **two calls per node**: each subproblem splits, so the number of nodes per level grows while each node's own subproblem gets smaller. Count the leaves, not the levels — the tree has $\Theta(n)$ leaves, and every node costs $\Theta(1)$, so the total is $\Theta(n)$. When in doubt, run substitution: guess $T(n)=cn$, check that $c(n/3)+c(2n/3)+1 = cn+1 = O(cn)$ holds, which confirms the linear bound.

```java
// Quicksort worst case, demonstrated: already-sorted input + "first element" pivot
// = Θ(n²). Randomised pivot makes it Θ(n log n) expected on every input,
// which is why every production sort (including C++ std::sort → introsort
// → heapsort fallback) refuses to keep this behaviour.
```

**Non-master-theorem reasoning you should be able to do by eye:**

- **Per-level reasoning:** "each of the $n$ elements is processed once per recursion level, and there are $\log n$ levels" → $O(n \log n)$.
- **Per-element reasoning:** "each element is pushed once and popped at most once" → $O(n)$ (BFS, topological sort).
- **Work conservation:** "total work = $n$ elements × constant" → $O(n)$ no matter how many levels, because the work per level *shrinks* (divide and conquer on the count of elements, e.g. `countInversions` by halving: $\sum n/2 + n/4 + \dots = O(n)$).

### 6. Amortised analysis: promises about sequences

Amortised analysis answers: "if I do $n$ operations, what is the **total** cost?" It is a *deterministic* statement about a sequence — no probabilities.

| Method | Idea | Example |
|--------|------|---------|
| **Aggregate** | total cost over $n$ operations, divided by $n$ | Dynamic array: $n$ appends copy $< 2n$ elements total → $O(1)$ amortised |
| **Accounting (banker's)** | charge each operation a little more than it costs; the surplus is stored as "credit"; spend credit on the expensive ops | Same: charge 3, spend up to $n$ on a resize → net $O(1)$ |
| **Potential** | define a potential $\Phi$ over the state; amortised cost = actual + $\Delta\Phi$; bound the total potential change | $T(n) = T(n/2) + \Theta(n) = \Theta(n)$ via $\Phi = \log n$ |

The canonical example, because it explains why `ArrayList.add` and `vector::push_back` are "constant time":

```
capacity:  1 → 2 → 4 → 8 → ... → n
copied:      1 + 2 + 4 + ... + n/2  <  n          geometric series
```

Total copying for $n$ appends is $< n$. The *individual* append that triggers a resize is $\Theta(n)$ — amortised analysis does not deny that, it just shows you can afford it.

**Three things amortised analysis is not:**

1. **Not worst case.** `ArrayList` has a real $O(n)$ insert at position 0. If your workload is "insert at the front", amortised $O(1)$ append tells you nothing.
2. **Not average case.** Amortised is worst-case-total: even the adversarial sequence has total $O(n)$.
3. **Not a licence to be lazy.** A `HashMap` resize is one $O(n)$ pause in the middle of an $O(1)$-looking program — exactly the kind of latency spike that makes a service look fine in the average and terrible at p99.

The other structure people quote amortised numbers for:

| Operation | Amortised | Worst | Note |
|-----------|-----------|-------|------|
| `ArrayList.add` / `push_back` | $O(1)$ | $O(n)$ | the resize |
| `ArrayDeque` add/remove both ends | $O(1)$ | $O(1)$ | genuinely worst-case too |
| `HashMap` put/get | $O(1)$ avg | $O(n)$ | collisions degrade to a list; also the *resize* |
| `HashMap` with adversarial keys | $O(n)$ | $O(n)$ | the "HashDoS" family — never hash user-controlled keys naively |
| DSU `find` + `union` | $O(\alpha(n))$ | $O(\log n)$ | $\alpha$ is the inverse Ackermann function: ≤ 4 for any $n$ you will ever build |

### 7. Space complexity, and the space–time trade

- **Auxiliary space** = extra memory beyond the input **and** the output. A function that returns a new array of size $n$ uses $O(1)$ auxiliary space, not $O(n)$ — a distinction that decides whether a memory limit passes.
- **Recursion depth is stack space.** A recursive traversal that can reach depth $n$ uses $O(n)$ stack *and* may die with `StackOverflowError` in Java ([`02-java-for-dsa.md`](02-java-for-dsa.md) §8) or blow the real stack in C++. The iterative version is $O(1)$ space — a different class, not just cleaner code.
- **Divide and conquer: depth is $O(\log n)$, total live memory can be $O(n)$.** Merge sort that allocates a fresh buffer per level is $O(n \log n)$ space; with one shared buffer it is $O(n)$.
- **A frame that allocates kills the analysis.** `int solve() { int[] tmp = new int[n]; ... return recurse(); }` is $O(n^2)$ space, not $O(n)$ — the classic recursion-with-array-copy trap.
- **Memoisation is the deliberate trade.** Top-down DP converts $O(2^n)$ time into $O(\text{states} \cdot \text{transition cost})$ time by spending $O(\text{states})$ space. But it does **not** flatten the stack: the recursion still nests one frame per state along the longest chain, so a $10^6$-state DP written top-down can still throw `StackOverflowError` in Java. The stack stops being the problem only when you switch to **bottom-up** DP (fill one table in increasing state order) or when the state graph's longest chain is itself shallow — $\Theta(\log n)$ for the divide-and-conquer-shaped DPs, which is why binary search and merge sort are safe recursively and a naive interval DP is not.
- **Space is a *time* cost too.** Allocation pressure triggers GC; the "cost per element" of a $O(n)$ list of boxed `Integer` is not the same as an `int[]` — see [`02-java-for-dsa.md`](02-java-for-dsa.md) and [`03-cpp-for-dsa.md`](03-cpp-for-dsa.md).

### 8. Lower bounds: proving you cannot do better

A lower bound is the only thing that turns a fast algorithm into a *finished* problem. The two that show up constantly:

**Comparison sorting: $\Omega(n \log n)$.** Proof sketch — a sorting algorithm that only compares elements is a decision tree; the input has $n!$ permutations; a binary decision tree of depth $d$ distinguishes at most $2^d$ cases, so $d \ge \log_2(n!) = \Omega(n \log n)$. Mergesort matches it, so $\Theta(n \log n)$ is optimal **for comparison-based sorting**.

That "for comparison-based" is the escape hatch, and it is how counting sort and radix sort break the bound: they use the *values*, not only comparisons.

**Searching an unsorted array: $\Omega(n)$** — you must look at every element, because any unseen element could be the one you are looking for. **Searching a sorted array: $\Omega(\log n)$** — the decision-tree argument again, this time with $n$ leaves.

**The habit worth building:** when a problem is rated "hard", ask "what is the lower bound here, and does my solution meet it?" If your solution is $O(n \log n)$ on a problem with an $O(n)$ solution, the gap *is* the hard part. Finding the gap is the skill; the notation is bookkeeping.

### 9. When Big-O lies to you

| Lie | What really happens | Counter-intuition to hold |
|-----|--------------------|--------------------------|
| "$O(\log n)$ beats $O(n)$" | At $n = 10^7$, a binary search is ~23 random memory probes; a linear scan is 10⁷ sequential reads that the prefetcher hides | Sequential access is nearly free; random access is ~100× more expensive per element. Constants are the *dominant* term until the asymptotic gap exceeds them. |
| "Amortised $O(1)$ is $O(1)$" | `HashMap` resize, `ArrayList` grow: one $O(n)$ stall | Amortised bounds are about totals. Latency percentiles live in the worst case. |
| "Hash lookup is $O(1)$" | Expected, not worst; $O(n)$ on collisions; `Integer` boxing makes the constant much worse than `int` | The *expected* qualifier is doing real work. Adversarial keys break it deliberately. |
| "Sorting is $O(n \log n)$, so it is fine" | At $n = 10^7$ it is ~2.3 × 10⁸ comparisons — seconds, not milliseconds | Convert to *operations* and compare against your time limit before trusting the label. |
| "A linked list uses less memory than an array" | Per node: value + next pointer + object header + padding. A `LinkedList<Integer>` node is ~40 bytes for a 4-byte value | Contiguity is a memory-layout decision with a constant factor of ~5–10×, plus a locality win. |
| "I only run this on small inputs" | Then constants dominate and Big-O is not the right tool | For $n < 10^4$, measure; do not analyse. |
| "It is $O(n)$" (unbounded $n$) | Inputs are integers/floats, and arithmetic is not unit-cost | Multiplication on 1000-bit numbers is not one operation. Bit complexity is a different subject. |

**The uncomfortable, correct conclusion:** complexity analysis is a *filter for large $n$*, and the crossover point depends on your language, machine, and memory layout. This is why "which is faster, array scan or binary search on this array?" has no theoretical answer — it has a measurement.

### 10. Measuring honestly — the other half of the skill

An $O()$ label is a hypothesis. The measurement loop is four steps:

1. **Fix the workload.** Generate the input once, store it, reuse it. Reading a file inside the timed loop measures your disk.
2. **Time the whole binary, not the function** — but subtract the harness. In C++: `/usr/bin/time -v ./a.out < in.txt`; in Java: `System.nanoTime()` around the algorithm, with the JVM already warmed.
3. **Warm up (JVM) and compile (native).** Java JIT: the first 10⁴ iterations measure the interpreter. C++: `-O0` vs `-O2` is a 2–10× difference that has nothing to do with your algorithm.
4. **Infer the exponent by doubling.** The ratio test is the actual analysis:

$$T(2n) \approx 2^{k}\,T(n) \quad\Longrightarrow\quad k = \log_2\!\left(\frac{T(2n)}{T(n)}\right)$$

```bash
# the whole measurement loop, in one shell
for n in 10000 20000 40000 80000 160000; do
  ./a.out "$n" > /dev/null
done
# time each, then compute log2(t(2n)/t(n)): ~1 → linear, ~2 → quadratic,
# ~log2(2) + log2(n) growth → n log n, ~constant → suspiciously constant-time
```

**Typical order of magnitude** (modern laptop, $n = 10^6$ random `int`s — *measure, do not trust this table*):

| Work | C++ `-O2` | Java (warmed) | Java boxed | Python |
|------|-----------|---------------|------------|--------|
| sum of 10⁶ `int`s | ~1 ms | ~1–3 ms | ~30–60 ms (`Integer`) | ~40 ms (builtin `sum`) |
| `sort` | ~80 ms | ~300 ms | ~2 s (`Integer[]`) | ~400 ms (`sorted`) |
| 10⁶ `HashMap` puts | ~200 ms | ~200 ms | ~400 ms | ~700 ms (`dict`) |
| nested 10³ × 10³ | ~1 ms | ~2 ms | — | ~30 ms |

The **10–25× gap between `sort(int[])` and `sort(Integer[])`** is the single most useful row in this table: it is not a different algorithm, it is allocation and indirection — the constant factor you ignored because Big-O let you.

### 11. Decision rules: given the constraint, what do I build?

First convert the constraint into a budget. At roughly $10^8$ simple operations/second in C++ and $3 \times 10^7$ in warmed Java, a 2-second limit is about:

| $n$ | Budget | What fits |
|-----|--------|-----------|
| $10^2$–$10^3$ | $10^6$–$10^7$ | anything, including $O(n^2)$ and $O(2^{n/2})$ with pruning |
| $10^4$–$10^5$ | $10^8$–$10^{10}$ | $O(n \log n)$ yes; $O(n^2)$ borderline at $10^5$ — measure |
| $10^5$–$10^6$ | $10^{10}$+ | $O(n)$ or $O(n \log n)$ required; $O(n^2)$ is out |
| $10^7$+ | — | $O(n)$ minimum; think about memory bandwidth and I/O, not arithmetic |

Then the structural rules that follow from the table in [`05-data-structure-selection-guide.md`](05-data-structure-selection-guide.md):

| Situation | Reach for | Why |
|-----------|-----------|-----|
| Values bounded by $k$ where $k \le$ few × $n$ | counting / bucket sort: $\Theta(n + k)$ | Beats $n \log n$; but memory is $O(k)$ |
| Many "find the value near $x$" queries | sort once + binary search: $O(n \log n)$ total | The universal fallback when you cannot see a structure |
| Repeated lookups of a small domain | direct-address table / array | $O(1)$, and better cache behaviour than a hash |
| Key order matters | ordered structure (BST, `TreeMap`, sorted array) | Hash gives you no order; sorting at the end is $O(n \log n)$ anyway |
| Order matters *and* the set changes | balanced tree / heap | — |
| Many "k-th smallest so far" | heap of size $k$: $O(n \log k)$ | Note: $O(n \log k)$, not $O(n \log n)$ |
| Offline queries, no interactivity | **sort by the query parameter, sweep** | Turns online difficulty into a single sort |
| Subarray sum/OR/AND queries, no updates | prefix sums / sparse table | $O(1)$ or $O(\log n)$ per query |
| Point update + range sum | Fenwick tree (BIT) | $O(\log n)$ both, $O(n)$ memory, ~15 lines of code |
| Range update + range sum | BIT (two trees) or lazy segment tree | — |
| Non-commutative range ops (min/max/max-subarray) | segment tree | BIT cannot do order-dependent ops |
| Dynamic connectivity | DSU | $O(\alpha(n))$; with offline queries, add rollback |

**Rule 0, which beats every table above:** *the input does not change and you only need the answer once — sort it, then use the simplest structure that answers the question.* A surprising share of "hard" problems are this, wearing a disguise.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---------|---------|-----|-----|
| Quoting $O()$ where $\Theta()$ is meant | "Optimal" solution turns out 10× slower than the classic | An upper bound is a permission slip | Prove the matching lower bound, or say "upper bound only" |
| Wrong $n$ | An "optimal" $O(n)$ solution times out | $n$ was elements when the input is bits, or lines when it is characters | Write "$n$ = ___" before any algebra |
| Counting a nested loop as $O(n^2)$ automatically | Over-engineering a problem that is $O(n)$ | Inner range may shrink, break early, or be smaller than $n$ | Count element visits, then sum |
| "Nested loop ⇒ $O(n^2)$" as a rule | Missing the two-pointer $O(n)$ pattern | The inner pointer never rewinds | Ask: does the inner index ever move backwards? |
| `list.contains()` / `indexOf` treated as $O(1)$ | $O(n^2)$ that passes small tests | Java list membership is a linear scan | `HashSet` / `HashMap` ([`02-java-for-dsa.md`](02-java-for-dsa.md)) |
| String comparison counted as $O(1)$ | Surprising $\log$ factor or an inner $O(k)$ loop | Comparing two strings costs their length | Count the comparison's cost, or hash/intern once |
| Amortised treated as worst case | p99 latency spike in production | Amortised bounds are about totals over a sequence | State worst case where latency matters |
| Expected treated as guaranteed | Adversarial input blows the bound | Expected = over the algorithm's randomness, and hash order depends on input | Randomise the key/hash when keys are untrusted |
| Counting recursion as $O(\log n)$ space | `$O(n^2)$` memory blowup | Frames that allocate arrays; or depth really is $O(n)$ | Count the frame's *contents*, not the frame |
| Buffer per recursion level in merge sort | Memory limit exceeded | $O(n \log n)$ space instead of $O(n)$ | One shared buffer + a temp array for the return |
| String `+=` in a loop | $O(n^2)$ TLE | Each `+` copies the whole string | `StringBuilder` / `StringBuffer` |
| `substring` assumed free | $O(n^2)$ in Java 7+ | `substring` **copies** the characters (unlike Python) | `char[]`/`offset+length`, or build once |
| Forgetting the output size | "auxiliary space" is wrong | Output is not counted as auxiliary, but you still must allocate it | State both auxiliary and total |
| Benchmarking with I/O inside the loop | 10× noise, wrong conclusion | Disk/parse time swamps the algorithm | Generate input once, time only the algorithm |
| Benchmarking a cold JVM | 5–20× slower than reality | Interpreted/C1 code, not JIT-compiled | Warm up 10⁴+ iterations before timing |
| Comparing against a $O(n)$ scan at $n = 10^4$ | Conclusion: "Big-O is wrong" | Constants dominate below ~10⁵ | Measure both; the crossover is real |
| Recursion depth $n$ in Java | `StackOverflowError` (an `Error`, not catchable) | Stack is bounded and small | Iterate, or add an explicit stack/queue |
| Unsorted binary search | Wrong answer, not just slow | The $\log n$ assumes sorted input | Sort first ($O(n \log n)$ total) |
| `O(2^n)` written where the code is $O(n 2^n)$ | Underestimate by a factor of $n$ | The per-step copy is the real cost | Include the body of the loop, not the recursion |
| Assuming more RAM is available | Answer rejected on memory limit | Stated limits are often 256 MB, not 16 GB | Count bytes: an `int[]` is 4n, a boxed list ~40n |
| Ignoring `n` in the *output* | Streaming solution "too slow" | Printing $n$ lines is $\Theta(n)$ even for an $O(\log n)$ algorithm | Count output in the total |

---

## 📊 Complexity cheat sheet

```
ITERATIVE
single pass                  O(n)            nested same-range        Θ(n²)
two pointers (no rewind)     O(n)            nested shrinking         Θ(n²)/2 work
prefix sum build             O(n)            break-early nested       Θ(steps)
build + n queries            O(n) + O(q)     hash per op              O(1) avg / O(n) worst

RECURSIVE (T = aT(n/b) + f(n))
binary search   T=T(n/2)+Θ(1)        Θ(log n)        merge sort  2T(n/2)+Θ(n)     Θ(n log n)
quicksort avg   T=T(k)+T(n-k)+Θ(n)  Θ(n log n)      quicksort worst             Θ(n²)
quick select    T=T(n/2)+Θ(n)       Θ(n)            counting/radix             Θ(n + k) / Θ(d(n+b))
linear walk     T(n)=T(n-1)+Θ(1)   Θ(n)            Akra-Bazzi (irregular b)   O(n^p)
```

```
STRUCTURES
array/dynamic array   O(1) get, O(1) amort add      hash map     O(1) avg, O(n) worst, unordered
stack / deque         O(1) push/pop both ends       balanced BST O(log n) + ordered
heap (binary)         O(log n) push/pop, O(1) peek  DSU          O(α(n)) amort
Fenwick tree          O(log n) update + query       seg tree     O(log n), arbitrary + lazy
prefix sums           O(n) build, O(1) query         sparse table O(n log n) build, O(1) query
```

```
GRAPHS
BFS / DFS (adj list)  O(V + E)     Dijkstra (bin heap)  O((V + E) log V)
BFS / DFS (matrix)    O(V²)        Dijkstra (fib heap)   O(E + V log V)
topological sort      O(V + E)     union-find            O(α(n)) amort
```

```
TIME BUDGET (approximate, per second)
C++ -O2        ~10^8 ops      Java (warmed)  ~3 × 10^7      Python  ~10^7
n = 10^3  →  10^6 ops   → anything
n = 10^5  →  10^10 ops  → O(n log n) max, O(n²) borderline
n = 10^6  →  10^12 ops  → O(n) required
n = 10^7  →  10^14 ops  → O(n) minimum; memory-bandwidth bound, not arithmetic
```

---

## 🧪 Practice Tasks (no answers here — attempt first)

1. **Three questions, three times.** Take any three solutions you have already submitted. For each: state what $n$ is, state the cost of the innermost comparison, and identify the dominant term. Find at least one where your *original* analysis was wrong.
2. **Growth measurement.** Write a program that runs the same algorithm on $n$ and $2n$ for $n = 10^4 \ldots 10^6$, prints the timing table, and prints $\log_2(t(2n)/t(n))$ for each row. Then deliberately implement an $O(n^2)$ version and check that the inferred exponent converges to 2. **Do this in both Java and C++** and explain the constant-factor gap in one paragraph.
3. **Counting pairs, twice.** Solve "count pairs $i < j$ with $a_i < a_j$" in $\Theta(n^2)$, then in $\Theta(n \log n)$ with a BIT, then in $\Theta(n + k)$ with a counting sort when values are bounded. Report time and space for all three, plus the crossover $n$ where the third version wins.
4. **Master theorem drill.** For each recurrence, state $a$, $b$, the critical exponent, the applicable case, and the result — then verify with a real implementation: $T(n)=3T(n/4)+\Theta(n)$; $T(n)=2T(n/2)+\Theta(\sqrt n)$; $T(n)=T(n/3)+\Theta(n)$; $T(n)=T(n-1)+T(\lfloor n/2\rfloor)+\Theta(1)$; $T(n)=4T(n/2)+\Theta(n^2)$. For the last one, derive the answer without the master theorem.
5. **Amortised proof, not assertion.** Prove $O(1)$ amortised append for a doubling dynamic array twice: once with the accounting method (state the charge and the credit balance) and once with the potential method (state $\Phi$). Then find an input that makes a *single* append cost $\Theta(n)$ and time it.
6. **Space audit.** Take five of your past solutions and report *auxiliary* space (excluding the output), including the recursion stack, in bytes at $n = 10^6$. Find the one with the worst space complexity and rewrite it to reduce it, measuring peak memory (`/usr/bin/time -v`, or `Runtime.totalMemory()`).
7. **Lower-bound hunt.** Find three problems where an $O(n)$ (or $O(n \log n)$) solution is provably optimal, and justify the lower bound in three sentences each. Then find one problem rated "medium" on a judge where a *counting-sort / bitset* trick breaks the $O(n \log n)$ barrier.
8. **Crossover experiment.** Implement array scan vs binary search on a sorted random `int[]` for $n = 10^3 \ldots 10^7$. Find the exact $n$ where binary search starts winning. Explain the result using cache-line size and prefetching. **This task is the point where Big-O stops being an abstraction.**
9. **Big-O lies, documented.** Find three real cases where a supposedly-slower algorithm beat a supposedly-faster one on this machine. For each, name the constant, the memory layout, or the hidden worst case responsible. Keep the list; it is your personal "when to distrust" reference.
10. **Amortised vs p99 (production framing).** Write a Java program that does 10⁶ `HashMap` puts and records the *maximum* single-operation latency, not the average. Compare with the mean. Then explain in five sentences why an SLO written in averages would be a bug — this is the bridge from DSA to [`../../software_architecture/README.md`](../../software_architecture/README.md).

---

## 🔗 Tech Cross-References (Section K)
- **Databases** — a B-tree index is $\Theta(\log_{b} n)$ with $b$ ≈ 200–1000 (fewer levels than a binary search tree, and the reason deep indexing is possible at all), a hash index is $O(1)$ expected / $O(n)$ worst, and a sequential scan is $O(n)$. The query planner chooses between them with exactly these numbers plus real cardinalities. "Do I add an index?" is a complexity question → [`../databases/README.md`](../databases/README.md)
- **Linux / CLI** — measurement is the other half: `/usr/bin/time -v` for peak RSS, `perf stat` for cache-miss and branch counts, `head -c` to size an input file. The ratio test in §10 is a shell loop — see [`../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md) and [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md)
- **Java** — where the constants come from: boxing, `ArrayList` growth, JIT warm-up, and the fact that a managed runtime makes *amortised* reasoning about GC pauses necessary → [`02-java-for-dsa.md`](02-java-for-dsa.md)
- **C++** — where the constants disappear: contiguous `vector`, templates with no boxing, and the same asymptotics with 10× smaller constants. Also the language where UB can make your "$\Theta(n \log n)$" quietly lie → [`03-cpp-for-dsa.md`](03-cpp-for-dsa.md)
- **ML / MLOps** — training cost is a complexity expression you can reason about before running: $\Theta(\text{params} \times \text{tokens})$; inference latency is dominated by memory bandwidth (the same sequential-vs-random trade-off); batch size vs memory is the classic time/space trade → [`../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md)
- **Economics & Business** — the same curve, different domain: marginal cost, economies of scale, and network effects are statements about how cost/benefit scales with $n$. A bridge note connecting $O(n)$ to economies of scale is listed in the folder README → [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md)

## 📝 Research Journey
- **Why:** I could state that quicksort is $O(n \log n)$ and quickselect is $O(n)$ without knowing which one to write, or *why* one is better. I was treating Big-O as trivia to memorise per algorithm instead of as a cost function I can compute for a problem I have not seen. Every hard problem I failed in the past came down to a wrong $n$ or a hidden $O(n)$ inside a loop.
- **Struggle:** Amortised analysis. For a long time "amortised $O(1)$" and "worst case $O(n)$" felt like a contradiction — either it is constant time or it is not. The fix was doing the accounting-method proof on paper for a doubling array and watching the credit balance pay for the resize. The second struggle was accepting that $\log_2(n!)$ gives a *lower* bound for comparison sorts and nothing else; until then I had no idea how radix sort escapes it.
- **Aha moment:** Writing the budget table in §11 by converting $n$ into *operations* ($10^8$/second), then asking "how many operations does this problem get to spend?" — that single reframing turned "is this enough?" from a guess into arithmetic, and it is the same question the course is actually asking.
- **Career link:** Every architecture decision is this note applied to a larger system. Index or no index, batch size, cache or recompute, queue or synchronous call, model size vs latency budget — each is a complexity argument that either holds at scale or quietly becomes an incident. And the honest measurement loop in §10 is the bridge between "the theory says" and "the dashboard says".

## 🔗 Related Topics
- [README.md](README.md) — the folder anchor
- [01-learning-roadmap.md](01-learning-roadmap.md) — where this toolkit sits in the phased plan
- [05-data-structure-selection-guide.md](05-data-structure-selection-guide.md) — the companion: complexity tells you the price, this tells you what to buy
- [02-java-for-dsa.md](02-java-for-dsa.md) — the per-operation cost table in the course language
- [03-cpp-for-dsa.md](03-cpp-for-dsa.md) — the same operations with 10× smaller constants
- [`../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md) — the measurement loop as shell commands
- [`../databases/README.md`](../databases/README.md) — indexes are complexity analysis with persistence
- [SUGGESTED] `06-recursion-to-iteration-mental-model.md` — bridges tree/graph traversal and both languages' stack limits (already in the folder backlog)
- [SUGGESTED] `07-probabilistic-methods.md` — expected vs amortised vs average, hashing, Monte Carlo counting, and the birthday paradox
- [SUGGESTED] `bridge-complexity-and-scaling.md` — bridge note: $O(n)$ in code vs economies of scale in business, the same curve in two fields

## 🤔 Open Questions
- [ ] Is the course graded on proofs of correctness as well as complexity? That decides whether the lower-bound material in §8 is exam material or a reading habit.
- [ ] Which online judges are strict enough that constants actually matter? (LeetCode is not; Codeforces sometimes is.) A list would make Practice Task 8 more meaningful.
- [ ] Should we build a small local benchmark harness in the code repo (`bench/`) so every experiment in §10 is one command instead of a hand-written loop?
- [ ] Amortised vs rolling-average latency for the software-architecture notes: is there a shared vocabulary I should standardise now?
- [ ] Bit complexity: at what problem size does "assume unit-cost integers" become a *wrongness* problem rather than a performance problem?

---
*Created: 2026-09-28 — measurement discipline note for the DSA track. Benchmarks, solution code, and their write-ups live in the code repo.*

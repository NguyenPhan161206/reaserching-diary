# Java for DSA — The Course Language

**Date:** 2026-09-28
**Status:** Draft
**Difficulty:** Beginner → Advanced (Java-specific)
**Template:** A (Tech Note) — adapted
**Tags:** `#DSA`, `#Java`, `#Collections`, `#Generics`, `#Performance`
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- Java is the **course language**: it is the language you are graded in, so every problem is attempted here first. C++ is the port target ([03-cpp-for-dsa.md](03-cpp-for-dsa.md)).
- The Collections Framework is powerful **and** dangerous: `HashMap` has no ordering guarantee, `ArrayList.get()` is $O(1)$ but `contains()` is $O(n)$, and **Java has no built-in monotonic stack / min-max queue**. You must write them.
- `PriorityQueue` exists and is a real binary heap, but it is a **max-heap by default** and has **no `decrease-key`** — this changes how you write Dijkstra.
- Boxing is a hidden $O(n)$ tax. `Integer` in a list costs ~16 bytes vs 4 for `int` in an array; this is why `Integer[]` + `Arrays.sort` is slow.
- `int` is 32-bit and overflows silently. Many DSA answers need `long`. There is no `unsigned`, and no `__int128` — for huge intermediates you need `BigInteger`.
- Deep recursion throws `StackOverflowError` (an `Error`, not an `Exception`) and you **cannot easily raise the limit** — iterative traversal is sometimes *required*, not optional.

---

## 🧠 Detailed Notes

### 1. Why Java is the course language, and what that changes

The DSA course materials are Java, so the exam-shaped assessments will be Java. Practising in another language and translating under time pressure adds a step that measures *translation speed*, not *algorithmic skill*.

**Consequence for this track:** every problem is attempted in Java first. The C++ port is optional and selective (rule of thumb: 1 in 5, weighted toward heaps and number theory).

**What Java trains that other languages do not:**
- Working inside a **managed** memory model — you must reason about what the runtime does to your object graph, not about `free`.
- Reading a **`NullPointerException` stack trace** and finding the exact line in a frame.
- Choosing the right **collection from a 20+ member hierarchy**, which is a real engineering decision you will make for the rest of your career.

### 2. The Collections Framework: what to reach for

| Need | Use | Complexity | ⚠️ Trap |
|------|-----|-----------|--------|
| Dynamic array | `ArrayList<T>` | get $O(1)$, add amortized $O(1)$ | `contains`/`indexOf`/`remove(int)` are $O(n)$ |
| Fixed-size array | `int[]` | $O(1)$ all ops | Size fixed; `Arrays.asList` is a fixed-size *view* |
| Fast queue/stack | `ArrayDeque<T>` | $O(1)$ | `null` not allowed; never use legacy `Stack` |
| Hash map (counting, lookup) | `HashMap<K,V>` | avg $O(1)$, worst $O(n)$ | **No iteration order**; resize at load factor 0.75 |
| Hash set | `HashSet<T>` | avg $O(1)$ | same ordering caveat |
| Tree map (ordered, range queries) | `TreeMap<K,V>` | $O(\log n)$ | Comparator must be consistent; slower than hash |
| Min/max heap | `PriorityQueue<T>` | $O(\log n)$ per op | **Max-heap by default**; no `decrease-key` |
| Sorting | `Arrays.sort(int[])` | $O(n \log n)$ | `Arrays.sort(Integer[])` uses boxing → slower |

**Why this matters more than it looks:** the single most common Java DSA performance bug is using `ArrayList.contains()` inside a loop, turning an $O(n)$ algorithm into $O(n^2)$ — and it still *passes small tests*. The fix is `HashSet`:

```java
// O(n^2) — passes small inputs, fails at n = 10^5
for (int x : nums) {
    if (result.contains(x - k)) { /* ... */ }
    result.add(x);
}

// O(n) — the derived version
Set<Integer> seen = new HashSet<>();
for (int x : nums) {
    if (seen.contains(x - k)) { /* ... */ }
    seen.add(x);
}
```

This is a concrete example of a principle in the [roadmap](01-learning-roadmap.md): *deriving* beats *recalling*.

### 3. `PriorityQueue` — the heap you get, and the two surprises

```java
import java.util.PriorityQueue;

PriorityQueue<Integer> maxHeap = new PriorityQueue<>();                  // natural order → MAX-heap
PriorityQueue<Integer> minHeap = new PriorityQueue<>((a, b) -> a - b);   // reversed → MIN-heap
```

**Surprise 1 — it is a max-heap by default.** Almost every DSA problem (Dijkstra, merging $k$ lists, kth smallest) needs a **min**-heap, so the comparator is mandatory. Getting this wrong produces code that looks correct and returns garbage.

**Surprise 2 — no `decrease-key`.** This is a structural difference from C++'s `priority_queue`. In Dijkstra, the textbook algorithm decreases a key inside the heap; Java gives you no way to do that, so you use **lazy deletion**:

```java
// Java: lazy deletion — push duplicates, skip stale ones on pop
int[] dist = new int[n]; Arrays.fill(dist, Integer.MAX_VALUE);
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);  // [node, dist]
dist[src] = 0; pq.offer(new int[]{src, 0});

while (!pq.isEmpty()) {
    int[] cur = pq.poll();
    int u = cur[0], d = cur[1];
    if (d > dist[u]) continue;          // stale entry → discard
    for (int v : adj[u]) {
        if (d + w < dist[v]) { dist[v] = d + w; pq.offer(new int[]{v, dist[v]}); }
    }
}
```

Trade-off: up to $O(E)$ heap entries instead of $O(V)$, so memory grows with edges — fine for interview and course sizes, a real problem at scale. This is a good thing to write in a solution's Language Notes: *the language changed the algorithm's space complexity*.

### 4. No built-in monotonic structures — write them yourself

Java has no `MaxSlidingWindow` type. Both structures are just a deque used correctly:

```java
// Monotonic deque: O(n) sliding-window maximum
Deque<Integer> dq = new ArrayDeque<>();   // stores INDICES, decreasing by value
int[] maxInWindows(int[] a, int k) {
    int[] out = new int[a.length - k + 1];
    for (int i = 0; i < a.length; i++) {
        while (!dq.isEmpty() && a[dq.peekLast()] <= a[i]) dq.pollLast();  // dominated → drop
        dq.offerLast(i);
        if (dq.peekFirst() <= i - k) dq.pollFirst();                      // out of window → drop
        if (i >= k - 1) out[i - k + 1] = a[dq.peekFirst()];
    }
    return out;
}
```

**The mental model that makes it stick:** the deque holds *candidates*, and a new element **dominates** (is better and newer) any element that is smaller — so the smaller one can never be a future answer, because the newer better element outlives it. That single sentence derives the whole algorithm.

**Why Java is fine here:** `ArrayDeque` is an efficient circular array-based deque, so push/pop at both ends are genuinely $O(1)$.

### 5. `HashMap` behaviour you must not get wrong

| Behaviour | Reality | Consequence |
|-----------|---------|-------------|
| Iteration order | **Unspecified** and effectively arbitrary (hash-bucket order) | Never rely on it. To get sorted key order use `TreeMap`, or `keys.sort(null)` |
| Null keys | One `null` key allowed | `HashMap.get(null)` is legal; `Hashtable` forbids it |
| Resizing | Load factor **0.75**, rehashes at threshold | Pre-size with `new HashMap<>(expected * 4 / 3 + 1)` on huge inputs |
| `hashCode`/`equals` | Must both be overridden together | A `HashMap` of your own class silently fails if you only override one |
| Keys are mutable | A key mutated after insertion becomes **unfindable** | Use `Integer`/`String`, never a mutable object you will edit |

That last row is a real bug source: `HashMap<Character, Integer>` with a `StringBuilder` key works until you append to the builder.

### 6. Generics and type erasure

Generics are a compile-time convenience; the JVM sees raw types. That single fact explains three things:

```java
List<String> a = new ArrayList<>();
// List<String> b = new ArrayList<Integer>();   // COMPILE ERROR — good, checked at compile time
// Raw: List c = new ArrayList(); c.add(1); c.add("s");  // compiles, explodes at runtime
```

1. **Type safety is compile-time only** — a raw `List` loses it entirely.
2. **You cannot create a generic array directly**: `new List<String>[n]` is illegal because arrays are covariant and erased. The idiomatic fix is a cast, or (better for graphs) a flat primitive array:

```java
@SuppressWarnings("unchecked")
List<Integer>[] adj = new List[n];          // the standard workaround (cast)
for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();

// Better for large graphs: flat head/to/next arrays (competitive-programming style)
int[] head = new int[n], to = new int[2 * m], nxt = new int[2 * m];
```

3. **Primitive generics need wrappers**: `List<int>` is illegal, so `List<Integer>` boxes every element. On large inputs prefer a `long[]`/`int[]` over `List<Long>`.

### 7. Integer overflow — the silent correctness bug

`int` is 32-bit signed: max $2^{31}-1 = 2\,147\,483\,647$. Java does **not** promote on overflow; it wraps.

```java
int a = Integer.MAX_VALUE;       // 2147483647
a + 1;                           // -2147483648   (no exception, wrong answer)
```

Rules that prevent most WA (wrong answer) submissions:

- Any multiplication inside a solution → use `long`.
- Distances, DP values, and counting values → `long`.
- Read with `long` when the statement mentions $10^9$ constraints and you multiply.
- `Integer.MAX_VALUE` as the "unreachable" sentinel → use `Integer.MAX_VALUE / 2` to leave room for one addition, or use `long` and `Long.MAX_VALUE`.
- `1 << 31` is **negative**. For 32-bit unsigned patterns use `1L << 31` or `Integer.toUnsignedLong(x)`.
- Beyond 64 bits → `java.math.BigInteger` (note: it allocates, so use it only when the statement demands it).

### 8. Recursion depth and `StackOverflowError`

The default JVM thread stack is small (commonly 512 KB–1 MB, giving roughly 10⁴–10⁵ simple frames). Exceeding it throws `StackOverflowError`, which extends `Error` — so `catch (Exception e)` will **not** catch it.

```java
// Doesn't help — the stack is already gone
try { solve(arr, 0); } catch (StackOverflowError e) { /* never reached */ }
```

`ulimit -s` (Linux) does **not** control the JVM's Java stack. The practical mitigations are:

- **Convert to iteration.** Skewed trees, deep paths, and large DP states are exactly where this bites.
- Run the JVM with a larger thread stack only as a last resort:
  `java -Xss16m Main` (newer JVMs cap `-Xss`; use `-XX:ThreadStackSize=...` or many small threads).
- Keep frames small: fewer local arrays per call, avoid passing large state by value (Java always passes references, so this is mostly about object graphs, not copies).

C++ has the same problem for different reasons — a raw 8 MB OS stack, and no runtime check you can catch; it just corrupts or crashes. See [03-cpp-for-dsa.md](03-cpp-for-dsa.md).

### 9. Input and output — the most common WA cause

`Scanner` is roughly **10–20×** slower than a hand-rolled `FastScanner`. On problems with $n = 10^6$ tokens it is the difference between TLE and AC.

```java
class FastScanner {
    private final byte[] buf = new byte[1 << 16];
    private int ptr = 0, len = 0;
    private int read() throws IOException {          // -1 at EOF
        if (ptr >= len) { len = System.in.read(buf); ptr = 0; if (len <= 0) return -1; }
        return buf[ptr++];
    }
    int nextInt() throws IOException {
        int c, sign = 1, out = 0;
        do { c = read(); } while (c <= ' ' && c != -1);
        if (c == '-') { sign = -1; c = read(); }
        while (c > ' ') { out = out * 10 + (c - '0'); c = read(); }
        return out * sign;
    }
}
```

For output, build one `StringBuilder` and print **once** — `System.out.println` per line is an I/O call per line.

Also remember: **classes need `public static void main`** to run, and LeetCode supplies the harness, so you submit the method only.

### 10. Strings, characters, and `String.hashCode()`

| Operation | Cost | Note |
|-----------|------|------|
| `s.length()` | $O(1)$ | Java caches length |
| `s.charAt(i)` | $O(1)$ | |
| `s.substring(a, b)` | $O(b-a)$ | **Copies** in Java 7+ — not a view |
| `s + "x"` in a loop | $O(n^2)$ | Use `StringBuilder` |
| `new String(chars)` | $O(n)$ | copies; `String.valueOf(chars)` also copies |
| `list.indexOf(s)` on `List<String>` | $O(n)$ with hashing cost | Use `HashSet<String>` |

`String.hashCode()` is:
$$ h(s) = \sum_{i=0}^{n-1} s[i] \cdot 31^{\,n-1-i} \pmod{2^{32}} $$

A fixed polynomial — fast and well-distributed, but **not collision-resistant**, which is why `HashMap` still verifies with `equals`. It also means an attacker can craft colliding keys (the classic "HashDoS" family of attacks). Never use it for security.

### 11. Complexity cheat sheet (what each operation really costs)

```
ArrayList.get(i)          O(1)        HashMap.get(k)         O(1) avg
ArrayList.add(x)          O(1) amort  HashMap.get worst      O(n)  ← collision tree
ArrayList.remove(int i)   O(n)  ←!!    TreeMap.get(k)         O(log n)
ArrayList.contains(x)     O(n)  ←!!    PriorityQueue.peek/poll O(1)/O(log n)
ArrayDeque.pollFirst()    O(1)        Arrays.sort(int[])      O(n log n)
Integer boxing            +alloc      Arrays.sort(Integer[]) O(n log n) + boxing
```

The two `O(n)` rows marked `←!!` are the ones that silently turn correct algorithms into $O(n^2)$ ones.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `list.contains()` in a loop | TLE only on large inputs | `HashSet` |
| `new PriorityQueue<>()` assumed min-heap | Wrong outputs on heap problems | Provide the reversed comparator |
| Dijkstra without lazy-deletion check | Wrong distances on graphs with multiple paths | `if (d > dist[u]) continue;` |
| `int` overflow | Silent WA on `$10^9 \times 10^9$` cases | `long` for products, sums, distances |
| `new PriorityQueue<>()` popping into an "in order" assumption | Wrong on duplicate keys | Only use when you don't need stability |
| `HashMap` iteration assumed sorted | Wrong output | `TreeMap` or sort the keys |
| `StackOverflowError` uncaught | Crash with no useful message | Convert deep recursion to iteration |
| `String +` inside a loop | $O(n^2)$ TLE | `StringBuilder` |
| `Scanner` for input | TLE | `FastScanner` |
| `HashMap` with mutable keys | "Impossible" lookup failures | Immutable keys only |
| Declaring a class `Main` in LeetCode | Wrong filename / no entry point | Match the judge harness |

---

## 🧪 Practice Tasks (no answers here — attempt first)

1. Implement `myHashMap()` supporting `put/get/remove` with **average $O(1)$**, from scratch, no `java.util.HashMap`.
2. Implement a **min-heap** (`insert`, `peek`, `pop`) on a plain `int[]`, then use it to solve "merge $k$ sorted lists".
3. Implement a **monotonic deque** sliding-window maximum, then extend to windowed *second* maximum.
4. Solve "kth smallest in a matrix" twice: once with a `PriorityQueue` (Java) and once with a binary-search-on-value approach. Compare space and runtime.
5. Take any Dijkstra problem you have already solved in C++ and rewrite it in Java. Write down in the solution's **Language Notes** section exactly how the missing `decrease-key` changed your memory usage.
6. Write a `FastScanner`, verify it against `Scanner` on 10⁶ random integers, and time both.
7. Find three DSA problems where `int` overflow silently produces a wrong answer. Submit them, get WA, fix with `long`.
8. Write a recursive and an iterative version of "flatten a binary tree into a list". Trigger `StackOverflowError` with a 10⁶-deep list, then explain in three sentences why the iterative version is *required*, not merely nicer.

---

## 🔗 Tech Cross-References (Section K)
- **C++** — the same algorithms with different container guarantees; the port target for this track → [03-cpp-for-dsa.md](03-cpp-for-dsa.md). `priority_queue` *has* `decrease-key` via indexed heaps; `vector` has no boxing.
- **JavaScript** — the *web* track language; contrast `Map`/`Set` (insertion-ordered) with Java's `HashMap`, and note the absence of a heap → [`../web_basics/README.md`](../web_basics/README.md)
- **Databases** — B-trees and hash indexes are the production versions of Phase 6 structures; `HashMap` is the toy → [`../databases/README.md`](../databases/README.md)
- **ML / MLOps** — `HashMap` counting is the most common preprocessing step in frequency/embedding code → [`../../ai_ml/README.md`](../../ai_ml/README.md)
- **Software architecture** — JVM vs native runtime trade-offs (startup, GC pause, memory) are a real architecture decision → [`../../software_architecture/README.md`](../../software_architecture/README.md)
- **Git** — every solution and every bug fix is a commit; the Language Notes section is where the diffs get compared → [`../../git_github/README.md`](../../git_github/README.md)

## 🔗 Related Topics
- [README.md](README.md) — the folder anchor
- [01-learning-roadmap.md](01-learning-roadmap.md) — where Java fits in the phased plan
- [03-cpp-for-dsa.md](03-cpp-for-dsa.md) — the port target, with the same structures in C++
- [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md) — the CLI you use to run and time every solution
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — why a compile/run script needs `chmod +x`
- [`../databases/README.md`](../databases/README.md) — SQL queries are hash-join and B-tree work under the hood
- [`../../software_architecture/02-modular-monolith-vs-microservices.md`](../../software_architecture/02-modular-monolith-vs-microservices.md) — JVM vs Node runtime trade-offs are a backend design decision
- [SUGGESTED] `04-jvm-memory-model-and-garbage-collection.md` — where your `int[]` actually lives, and why GC pauses are an architecture concern
- [SUGGESTED] `05-java-oop-for-algorithmists.md` — the minimum OOP needed to read a course codebase (classes, interfaces, inheritance, `Comparable`)

## 🤔 Open Questions
- [ ] Java version target for the course: 8 (`var`, streams), 11, or 17+? Changes which APIs are fair game.
- [ ] Does the course allow `java.util.stream` in submissions, or must it be plain loops?
- [ ] Should `FastScanner` be copied from a template in the code repo so it is never retyped?
- [ ] Is there a course-specific judging time limit that would change the input strategy?

---
*Created: 2026-09-28 — language note for the DSA track. Code, solutions, and measurements live in the code repo.*

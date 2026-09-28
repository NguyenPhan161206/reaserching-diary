# C++ for DSA — The Interview & Contest Language

**Date:** 2026-09-28
**Status:** Draft
**Difficulty:** Intermediate → Advanced (C++-specific)
**Template:** A (Tech Note) — adapted
**Tags:** `#DSA`, `#C++`, `#STL`, `#Templates`, `#Memory`
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- C++ is the **port target**: it is where algorithmic intent is judged most directly (interviews, Codeforces). Java is attempted first ([02-java-for-dsa.md](02-java-for-dsa.md), [01-learning-roadmap.md](01-learning-roadmap.md)).
- Value semantics are the big difference from Java: `vector<int> v2 = v1` **copies**; `std::move` or `&` avoids it. Getting this wrong costs time and memory.
- `vector<T>` is not an array. It is a **contiguous dynamic array with a capacity that only grows**, so `push_back` is amortized $O(1)$ but `insert`/`erase` in the middle are $O(n)$.
- **Iterator invalidation** is the C++ bug Java does not have. Any `push_back` can invalidate iterators, pointers, and references — including in `unordered_map` on rehash.
- `priority_queue` *is* a min-/max-heap, and with an **indexed-heap trick** you can implement real `decrease-key` — the thing Java cannot do.
- Undefined behaviour is the C++ failure mode. Buffer overrun, dangling reference, and overflow do not throw; they produce silently wrong answers or a crash with no message.
- Fast I/O is one line: `ios::sync_with_stdio(false); cin.tie(nullptr);`

---

## 🧠 Detailed Notes

### 1. Why C++ is the port target, and what it adds

Java teaches you *managed* algorithmic thinking. C++ adds three things Java deliberately hides:

1. **Cost is visible.** You choose the container, so you choose the constant factor and the memory layout. Interviewers and contests care.
2. **Data structure freedom.** Java's Collections Framework is broad but closed; C++'s STL plus a few custom structs lets you build exactly the structure the problem needs — including a real `decrease-key` heap.
3. **Value semantics.** Objects live wherever you put them. That is powerful (no aliasing surprises) and dangerous (copies you did not intend).

**The rule for this track:** attempt in Java, then port to C++ **from memory, not by translating**. A line-by-line transliteration wastes the exercise — the goal is to notice which parts of your solution were language-specific.

### 2. Containers: the decision table

| Need | Use | Complexity | ⚠️ Trap |
|------|-----|-----------|--------|
| Dynamic array | `vector<T>` | index $O(1)$, push_back amortized $O(1)$ | `insert`/`erase` in middle $O(n)$; capacity doubles → reallocation |
| Fixed buffer | `array<T,N>` | $O(1)$ | Size is a compile-time constant |
| Hash map | `unordered_map<K,V>` | avg $O(1)$, worst $O(n)$ | Adversarial keys can force $O(n^2)$; `operator[]` **inserts** on miss |
| Sorted map / set | `map<K,V>`, `set<K>` | $O(\log n)$ | Slower; use only when order is required |
| Min/max heap | `priority_queue<T, vector<T>, greater<T>>` | $O(\log n)$ | Max-heap by default (same as Java) |
| Double-ended queue | `deque<T>` | $O(1)$ at both ends | Contiguous chunks, not one block |
| Stack / queue | `stack<T>`, `queue<T>` | $O(1)$ | Thin wrappers over `deque` — no iterator access |
| Ordered set ops | `set<T>` with iterators | $O(\log n)$ to find, $O(1)$ to erase by iterator | Erase by value is $O(\log n)$ |

**The contest trap that catches everyone:**

```cpp
std::unordered_map<int, int> freq;
for (int i = 0; i < n; i++) freq[arr[i]]++;   // fine

int k = freq[missingKey];                      // DANGER: operator[] INSERTS k with value 0
if (k == 0) /* looks like "not present" */ ;   // ...and the map just grew by one entry
```

Use `find` / `count` when absence matters:

```cpp
auto it = freq.find(missingKey);
if (it == freq.end()) { /* absent */ }
else { /* it->second */ }

// or in C++20, one-liner:
if (auto it = freq.find(k); it != freq.end()) { /* ... */ }
```

### 3. `vector` internals: capacity, reallocation, and why it matters

`vector<T>` keeps a pointer, a size, and a **capacity**. `push_back` when size == capacity triggers reallocation, typically to ~2× capacity, then moves (or copies) the elements.

```cpp
std::vector<int> v;
for (int i = 0; i < 1'000'000; i++) v.push_back(i);   // ~20 reallocations, total ~2M element moves
```

- **Amortized $O(1)$** per `push_back`, not worst-case $O(1)$ — that is what "amortized" means here.
- **Do not keep pointers or references across a `push_back`** — they dangle after reallocation. This is the classic C++ crash:

```cpp
int* p = &v[0];
v.push_back(99);      // may reallocate
*p = 5;               // undefined behaviour if reallocated
```

- **Pre-size when you know the size** to avoid reallocation churn:
  `v.reserve(n);`
- `resize(n)` changes size (value-initialises new elements), `reserve(n)` changes capacity only.

### 4. Iterator invalidation — the bug Java cannot give you

| Operation | Invalidates |
|-----------|-------------|
| `vector::push_back`, `insert`, `erase`, `clear` | all iterators **and** references **and** pointers past the change point |
| `vector::reserve` (reallocates) | all iterators/references/pointers |
| `deque::push_back`, `push_front` | **all** iterators (references stay valid) |
| `list`, `forward_list` insert/erase | only the erased element's iterator |
| `map`/`set` erase | only the erased element's iterator |
| `unordered_map` insert that rehashes | **all** iterators; references to elements stay valid |

The safe pattern for a classic two-pointer loop:

```cpp
for (size_t i = 0, j = a.size() - 1; i < j; ++i, --j) {
    std::swap(a[i], a[j]);      // never holds an iterator across a mutation
}
```

And when removing while iterating an `unordered_map`, use the iterator-based `erase`:

```cpp
for (auto it = freq.begin(); it != freq.end();) {
    if (it->second < threshold) it = freq.erase(it);   // returns the next valid iterator
    else ++it;
}
// freq.erase(x) inside the loop is fine for unordered_map but invalidates
// the iterator it came from — the returned-next form is the habit to build.
```

### 5. A real `decrease-key` heap — what Java cannot give you

Java's `PriorityQueue` has no `decrease-key`, so Dijkstra there needs lazy deletion ([02-java-for-dsa.md](02-java-for-dsa.md)). In C++ you can implement the textbook version with an **indexed heap**: each node knows its position in the heap array, so a key update is a sift-up from a known index.

```cpp
class IndexedMinHeap {
    std::vector<int> heap_;      // heap positions -> node ids
    std::vector<int> pos_;       // node id -> heap position (-1 if absent)
    std::vector<int>& key_;      // current distance per node (owned by caller)

    void siftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (key_[heap_[parent]] <= key_[heap_[i]]) break;
            std::swap(heap_[parent], heap_[i]);
            pos_[heap_[parent]] = parent; pos_[heap_[i]] = i;
            i = parent;
        }
    }

public:
    IndexedMinHeap(int n, std::vector<int>& key) : pos_(n, -1), key_(key) { heap_.reserve(n); }

    bool empty() const { return heap_.empty(); }

    void update(int id, int newKey) {          // this IS decrease-key
        key_[id] = newKey;
        siftUp(pos_[id]);
    }
};
```

With this, Dijkstra holds at most $O(V)$ entries instead of $O(E)$, and relaxations that fail are rejected *before* they enter the heap.

**The lesson, not just the code:** the standard library's data structure is a *design choice with a cost model*. Knowing which cost you are paying (Java: memory, C++: your own code) is the actual skill.

### 6. Value semantics: copies, references, and moves

This is the single biggest behavioural difference from Java.

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = a;        // DEEP COPY — b is independent, costs O(n)
std::vector<int> c(std::move(a));  // MOVE — cheap, a is now empty
```

- **Function parameters:** by value copies; use `const T&` for read-only, `T&` for mutation, `T&&` with `std::move` to take ownership.
- **Return values are move-elided** in modern C++ — returning a local `vector` does not copy.
- **Rule of zero:** rely on the compiler-generated copy/move/destruct instead of writing any (the safest and fastest choice).

```cpp
void print(const std::vector<int>& v);       // no copy
void addOne(std::vector<int> v);              // copy on call — a common perf bug
void addOneInPlace(std::vector<int>& v);     // no copy
```

### 7. Templates — why the STL can be fast where Java boxes

`std::vector<int>` stores `int` **contiguously, unboxed** — 4 bytes per element, no object header, no per-element allocation. Java's `List<Integer>` stores 8-byte references to separate 16-byte `Integer` objects, scattered on the heap.

That is the whole explanation for the constant-factor gap, and it shows up in the standard cost table:

| Operation | Java | C++ |
|-----------|------|-----|
| Store 10⁷ ints | ~160 MB + cache misses | ~40 MB, one contiguous block |
| `sort` | `Integer[]` + boxing, or `Arrays.sort(int[])` (fast path) | `std::sort(int*)` inlineable |
| Function call per element | megamorphic dispatch possible | often inlined by the optimiser |

**Caution:** STL containers are fast partly because they are *tightly coupled* — the standard only requires them to be safe and correct, not optimally fast. Beyond a point, hand-rolled arrays win. That is exactly what Phase 6 "flat adjacency list" is about.

### 8. Undefined behaviour — the silent-wrong-answer engine

C++ does not check bounds, and overflow on signed integers is UB (in practice it wraps). The failure is a wrong answer or a crash with no diagnostic.

| Trap | Example | Safe habit |
|------|---------|-----------|
| Off-by-one | `a[i]` where `i == a.size()` | Iterate `for (int x : a)` or use `.size()` as the exclusive bound |
| Signed overflow | `int` product of two values near $10^9$ | `long long` for products, sums, distances |
| Dangling reference | reference to `v[0]` after `push_back` | re-index after mutation, or use indices not pointers |
| Use-after-free / double free | manual `new`/`delete` | prefer `std::vector`, `std::string`, smart pointers |
| Modulo of negatives | `(-7) % 3 == -1` in C++ (Python/Java give `2`) | normalise: `((x % m) + m) % m` |
| Uninitialised read | `int a[100];` then read | value-initialise `int a[100]{}` |
| `scanf` format mismatch | `%d` for a `long long` | use `cin`, or `%lld` |

**Sanitizers are the professional habit.** Compile once with them and the "mysterious WA" class of bugs mostly disappears:

```bash
g++ -std=c++17 -O1 -g -fsanitize=address,undefined main.cpp -o main_asan
./main_asan < input.txt
```

### 9. Fast I/O and the compile line

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);   // stop cout flushing per read
    cin.tie(nullptr);              // stop cout flushing per cin read
    int n; cin >> n;
    // ...
}
```

- Without `sync_with_stdio(false)`, `cin`/`cout` are locked to the C stdio layer — roughly **5–10×** slower.
- **Compile flags:** `-std=c++17` (the common judge default), `-O2` for speed, `-g -fsanitize=...` for debugging. Time it with `time ./a.out`.
- `endl` flushes every line — use `'\n'`.

### 10. Complexity cheat sheet (what each operation really costs)

```
vector[i]                    O(1)        unordered_map op       O(1) avg
vector.push_back             O(1) amort  unordered_map worst    O(n)  ← collision chain
vector.insert/erase at k     O(n)  ←!!   map / set op           O(log n)
vector.erase(iterator)       O(n)  ←!!   priority_queue top/pop  O(1) / O(log n)
deque push/pop both ends     O(1)        lower_bound             O(log n)
list insert/erase            O(1)        std::sort              O(n log n) (introsort)
pair<int,int> in a set       O(log n) *comparisons*, boxed
```

The starred line: `std::set<std::pair<int,int>>` is common in graph code and each comparison is a lexicographic double-compare — noticeable on hot loops.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `std::vector<int> b = a;` in a function | Unexpectedly slow | pass `const T&` |
| Keeping an iterator across `push_back` | Crash or silent corruption | use indices, or re-acquire |
| `map[key]` on a missing key | Map grows unexpectedly | `find`/`count` when absence matters |
| `priority_queue<>` assumed min-heap | Wrong output | `priority_queue<T, vector<T>, greater<T>>` |
| `int` overflow | Silent WA | `long long` for products and distances |
| Negative `%` | Off-by-one on modular problems | `((x % m) + m) % m` |
| Forgetting `ios::sync_with_stdio(false)` | TLE | add it as the first line of `main` |
| `endl` in a loop | TLE | `'\n'` |
| `sort(a.begin(), a.end(), greater<int>())` on `vector<int>` | — | works; or `std::greater` with the STL heap |
| Undefined behaviour, no message | Wrong answer, no crash | build with `-fsanitize=address,undefined` |
| Deep recursion on skewed trees | Stack overflow (uncatchable) | iterative traversal |

---

## 🧪 Practice Tasks (no answers here — attempt first)

1. Implement `std::vector<int>`-backed **dynamic array with amortized growth** from scratch, then prove (in writing) why `push_back` is amortized $O(1)$.
2. Implement an **indexed min-heap with `decrease-key`** (sketch above). Solve Dijkstra on a graph with 10⁵ nodes and compare memory against Java's lazy-deletion version.
3. Take one Java solution from the code repo and port it to C++ **from memory**. In the write-up's **Language Notes**, record every place the two languages disagreed about cost.
4. Write a program that triggers **iterator invalidation** and predict the output *before* running. Then run it under `-fsanitize=address` and explain the report.
5. Time `sort` of 10⁷ `int` in C++ versus 10⁷ `Integer` in Java. Explain the gap in terms of memory layout, not "C++ is faster".
6. Implement the **flat adjacency list** (`head[]`, `to[]`, `nxt[]`) for a graph with 10⁶ edges and compare it against `vector<vector<int>>` in both time and memory.
7. Find a problem where `int` overflow and **negative modulo** both bite, and write the corrected version with `long long` and normalised modulo.
8. Compile a solution with `-D_GLIBCXX_DEBUG` and run it; explain in one paragraph what the bounds checking buys you versus a sanitizer.

---

## 🔗 Tech Cross-References (Section K)
- **Java** — the course language and the port source; the two container models are the core contrast of this track → [02-java-for-dsa.md](02-java-for-dsa.md)
- **Linux** — you compile, time, and sanitise from the shell; `g++` flags, `time`, and `ulimit` are part of the workflow → [`../linux/README.md`](../linux/README.md)
- **Databases** — B-trees, hash indexes, and query planners are the production version of the structures here; the "flat array vs object" debate is literally database page layout → [`../databases/README.md`](../databases/README.md)
- **ML** — the same contiguity argument explains why tensors are flat buffers, and why `float32` beats a Python list of floats → [`../../ai_ml/README.md`](../../ai_ml/README.md)
- **Software architecture** — choosing a runtime (JVM vs native vs Go) is choosing where these trade-offs land → [`../../software_architecture/README.md`](../../software_architecture/README.md)
- **Git** — each solution and each bug fix is a commit; the Language Notes section is where the Java↔C++ diffs get compared → [`../../git_github/README.md`](../../git_github/README.md)

## 🔗 Related Topics
- [README.md](README.md) — the folder anchor
- [01-learning-roadmap.md](01-learning-roadmap.md) — where C++ fits in the phased plan
- [02-java-for-dsa.md](02-java-for-dsa.md) — the course language, the port source
- [`../linux/03-text-processing-and-pipes.md`](../linux/03-text-processing-and-pipes.md) — pipes for feeding test data into `./a.out`
- [`../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md) — moving between the two repos without a wall of output
- [`../databases/README.md`](../databases/README.md) — SQL query cost is these structures with persistence
- [`../../git_github/03-core-daily-workflow.md`](../../git_github/03-core-daily-workflow.md) — one commit per problem
- [SUGGESTED] `06-flat-arrays-and-memory-layout.md` — how cache lines, AoS vs SoA, and the flat adjacency list determine constant factors
- [SUGGESTED] `07-debugging-competition-code.md` — sanitizers, `gdb` basics, and reading a RE with `-O2` in it

## 🤔 Open Questions
- [ ] C++ standard target for the track: C++17 (judges usually cap here) or C++20?
- [ ] Should the code repo keep a `build.sh` with the exact judge flags (`-std=c++17 -O2`)? Reduces setup friction.
- [ ] Is the sanitizer habit worth a documented ritual, or just a note to self?
- [ ] Does the interview target (if any) use C++ or a language-neutral whiteboard — i.e. is this language note even needed for interviews?

---
*Created: 2026-09-28 — language note for the DSA track. Code, solutions, and measurements live in the code repo.*

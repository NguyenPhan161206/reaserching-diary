# 05 — Multithreading & Asynchronous Programming

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-11
**Tags:** `#Android`, `#Concurrency`, `#Threads`, `#Coroutines`, `#AsyncTask`, `#Mutex`, `#Retrofit`, `#REST`
**References:** Android Developers — "Processes and threads", "Coroutines on Android", `kotlinlang.org/docs/coroutines-guide.html`; `developer.android.com/guide/background`; Jake Wharton / Roman Elizarov talks on coroutines; Retrofit & OkHttp docs
**Status:** Draft
**Difficulty:** Intermediate → Advanced
**Prerequisites:** [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md), [`../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md)

---

## 📌 Key Takeaways (TL;DR)
- Android has **one main (UI) thread per process**, and it must stay free to draw frames and dispatch input. Blocking it → jank → **ANR** ("Application Not Responding" after ~5s).
- There are **two reasons to leave the main thread**: (1) do slow work *off* it, and (2) get back *onto* it to touch the UI. Every async API is a way to express those two facts.
- **Threads are the primitive; coroutines are the modern tool.** `AsyncTask` is deprecated. `Handler`/`Looper` is the underlying message-queue machinery, worth understanding, not worth writing.
- **Concurrency introduces shared-state hazards** (race conditions, visibility). A `Mutex` (or single-thread confinement) protects critical sections.
- **Networking is the canonical async task**, and Retrofit+OkHttp+coroutines is the standard stack — with built-in `suspend` support.

## 🧠 Mental Model
Picture the process with a **single UI thread holding a phone line to the screen**, and worker threads doing errands:

```
┌──────────────────────────────────────────────┐
│  Process                                       │
│                                                │
│  Main thread (UI) ──► draws frames, input       │  ← must NEVER block
│        │ message queue (Looper)                │
│        │                                       │
│  Worker thread 1 ──┐                           │
│  Worker thread 2 ──┼──► offload slow work      │
│  Worker thread N ──┘                           │
│        │                                       │
│        └── post results BACK to main thread ──► update UI
└──────────────────────────────────────────────┘
```

> **The whole discipline in one sentence:** *compute off the main thread, publish on the main thread.* Every API below — `Handler`, `AsyncTask`, `Executor`, `coroutines` — is a different way to say that.

**Frame budget math:** a smooth UI is **60 fps** (16.6 ms per frame) or **120 fps on high-refresh screens** (8.3 ms). If your work on the main thread exceeds that budget, frames drop — that's "jank". ANR is the extreme end: **5 seconds** of unresponsive main thread.

## 🔬 Detailed Notes

### 1. Concurrency, threads, and the main thread
- **Process** = your app (own memory, own sandbox).
- **Thread** = an execution path inside the process. All components (Activity/Service/Receiver) start on the **main thread** by default.
- **Concurrency** = interleaving multiple tasks; **parallelism** = truly simultaneous (needs cores). Phones have a few cores; the OS schedules many threads onto them.

**Why the main thread is special:** the `Choreographer` schedules frame rendering on it, and input events are dispatched on it. Block it and the UI freezes — even if the CPU is idle on other cores.

### 2. Creating & executing a `Thread`
```kotlin
// Raw Thread — the primitive. Note the two halves: work off-thread, result on-thread.
Thread {
    val data = download()                    // OFF main thread
    runOnUiThread {                           // BACK on main thread
        textView.text = data
    }
}.start()
```
Bare `Thread`s are fine for a quick lesson but bad in production: no lifecycle awareness (a thread can outlive the Activity → memory leak/`WindowLeaked`), no cancellation, no result plumbing, thread-per-task doesn't scale. That is why higher-level tools exist.

**The underlying machinery — `Looper`/`Handler`/`MessageQueue`:** each thread *can* have a `Looper` that pumps a FIFO `MessageQueue`. The main thread's Looper drives the UI. A `Handler` posts `Message`/`Runnable`s into a queue:
```kotlin
val handler = Handler(Looper.getMainLooper())
handler.post { textView.text = "updated on main" }     // schedule onto the main thread
```
**Mental model:** a `Handler` is "mail a task to this thread's inbox". `runOnUiThread` and `View.post` are sugar over it. Coroutines, below, are built on similar dispatch machinery.

### 3. Pros & cons of multithreading
| Pros | Cons |
|------|------|
| Keeps the UI responsive | **Race conditions** on shared mutable state |
| Uses multiple cores (parallelism) | **Deadlocks** (two threads waiting on each other) |
| Hides latency (I/O in parallel) | **Visibility** issues (stale caches without synchronization) |
| Overlaps compute + network | More memory, harder debugging, non-determinism |

> **The senior-engineer instinct:** parallelism is *not* automatically good on a battery-limited device. More threads = more context switching = more power. Often the right answer is **fewer threads doing more**, or cooperative single-threaded concurrency (coroutines).

### 4. Asynchronous programming in Android
**`AsyncTask` — deprecated, know it only for history/exams:**
```kotlin
// DEPRECATED — do not use in new code. Shown because courses still reference it.
class LoadTask : AsyncTask<String, Int, String>() {
    override fun doInBackground(vararg p: String): String = download(p[0])   // worker thread
    override fun onProgressUpdate(vararg values: Int) { bar.progress = values[0] } // main
    override fun onPostExecute(result: String) { textView.text = result }   // main thread
}
```
Why it died: leaks the Activity, hard to cancel, `execute`/`executeOnExecutor` confusion, no structured error handling. The tasks are: **do work off-thread, update progress, deliver result** — exactly what coroutines express better.

**Coroutines — the modern answer (structured concurrency):**
```kotlin
// Dispatcher decides WHICH thread; suspend = "may pause without blocking a thread".
lifecycleScope.launch(Dispatchers.Main) {
    val data = withContext(Dispatchers.IO) {   // switch to IO thread pool for blocking I/O
        repository.fetch()                      // suspend function
    }
    textView.text = data                        // back on Main automatically
}

// Parallel work with a result
viewLifecycleOwner.lifecycleScope.launch {
    val a = async(Dispatchers.IO) { fetchA() }
    val b = async(Dispatchers.IO) { fetchB() }
    val combined = a.await() + b.await()        // structured: parents wait for children
}
```
**What "suspend" really means:** a `suspend` function is a *state machine under the hood*. When it hits a real suspension point, it saves its state and **returns the thread to the pool** — no thread is blocked. That's the difference between a coroutine (cheap, thousands) and a thread (expensive, hundreds). This is the same idea as the JavaScript event loop vs. blocking I/O: cooperative concurrency beats thread-per-task.

**Structured concurrency & cancellation:** coroutines launched in a `scope` are cancelled when the scope dies (`lifecycleScope`, `viewLifecycleOwner.lifecycleScope`, `viewModelScope`). This is the fix for AsyncTask's leak: **the coroutine dies with the screen**, automatically.

**Dispatchers:**
| Dispatcher | Use for |
|------------|---------|
| `Dispatchers.Main` | Touch UI (the default in a `launch` from `lifecycleScope` if you specify Main) |
| `Dispatchers.IO` | Blocking I/O: network, disk, DB |
| `Dispatchers.Default` | CPU-heavy work: parsing, sorting, image processing |
| `Dispatchers.Unconfined` | Avoid unless you know why |

**Synchronization & `Mutex`:**
When multiple coroutines/threads touch shared mutable state, guard the critical section.
```kotlin
private val mutex = Mutex()
private var counter = 0

suspend fun incrementSafely() {
    mutex.withLock {          // suspend-based lock (coroutine-friendly, non-blocking-wait)
        counter += 1
    }
}
```
| Tool | When |
|------|------|
| **Confinement** (single thread / `Dispatchers.Main` or a single-thread dispatcher) | Simplest; no lock needed if only one thread touches the state |
| `Mutex` (coroutines) | Suspend functions sharing state |
| `synchronized` / `ReentrantLock` (threads) | Classic thread-shared state (blocks the thread) |
| `AtomicInteger` / `AtomicReference` | Single-variable read-modify-write (lock-free) |
| `Flow` / `StateFlow` | Sharing a *stream* of state safely (preferred for UI state) |

> **Classic race:** `counter++` is read-modify-write — two coroutines can both read `5`, both write `6`. `Mutex` or an `Atomic`/`MutableStateFlow` fixes it. The single best default is **immutable state + `StateFlow`**, so sharing is safe by construction.

### 5. Networking — REST APIs with Retrofit + OkHttp
**REST in one breath:** resources identified by URLs, manipulated with HTTP verbs (`GET`/`POST`/`PUT`/`DELETE`), carrying JSON, stateless. (Deep theory in [`../software_architecture/`](../../software_architecture/README.md).)

```kotlin
// 1) Define the interface — Retrofit generates the implementation
data class UserDto(val id: Long, val name: String, @SerialName("email") val email: String?)

interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Long): UserDto     // suspend → coroutine-friendly

    @POST("users")
    suspend fun createUser(@Body user: UserDto): UserDto
}

// 2) Build once (Singleton)
val okHttp = OkHttpClient.Builder()
    .addInterceptor(HttpLoggingInterceptor().apply { level = HttpLoggingInterceptor.Level.BASIC })
    .addInterceptor { chain ->                              // auth header example
        val req = chain.request().newBuilder()
            .header("Authorization", "Bearer $token").build()
        chain.proceed(req)
    }
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttp)
    .addConverterFactory(Json.asConverterFactory("application/json".toMediaType())) // kotlinx.serialization
    .build()

val api = retrofit.create(ApiService::class.java)

// 3) Call from a coroutine (Retrofit's suspend handles the threading switch for you)
lifecycleScope.launch {
    try {
        val user = api.getUser(42)
        textView.text = user.name
    } catch (e: IOException) {            // network failure
        showOffline()
    } catch (e: HttpException) {          // non-2xx
        showError(e.code())
    }
}
```
**Why this stack:** OkHttp handles connections/retries/HTTP2; Retrofit turns annotations into typed calls; `kotlinx.serialization`/Moshi converts JSON↔objects at compile time; `suspend` makes the whole call non-blocking and cancellation-aware. Together they satisfy "compute off main, publish on main" without you writing a single `Thread`.

**The professional pattern — the repository + offline-first:**
```
UI (ViewModel) ──► Repository ──► { Room (local, source of truth) , Retrofit (network) }
```
Read from Room and show instantly; refresh from the API in the background; write to Room then sync. This is notes 04 and 06 meeting in the middle.

## ⚖️ Trade-offs Summary
| Decision | Option A | Option B |
|----------|----------|----------|
| Async tool | Coroutines (structured, cancellable) | Executor/Thread (procedural, manual lifecycle) |
| Shared state | Immutable + `StateFlow` (safe) | Mutable + `Mutex` (needs discipline) |
| Parallelism | Coroutines/`async` (cheap) | Thread pool (heavy but predictable) |
| Networking | Retrofit+suspend (typed, idiomatic) | Raw OkHttp/`HttpURLConnection` (manual) |
| Error handling | `try/catch` in coroutine + `Result` | Global crash reporter only (too late) |

## 🏭 Industry Corners & Connected Tech
- **Coroutines ↔ the JS event loop:** both are *cooperative* concurrency; a `suspend` point is like an `await`. Compare [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md).
- **Threads & memory ↔ DSA/Java:** `HashMap` thread-safety, visibility, and `synchronized` come straight from the JVM model. [`../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md).
- **REST ↔ server architecture:** the API you consume is the server you studied in [`../web_server/`](../web_server/README.md) and [`../../software_architecture/`](../../software_architecture/README.md).
- **Offline-first sync ↔ network partitions:** retries, idempotency, and conflict resolution are distributed-systems concepts.
- **Battery:** wakeups and radio use are the mobile-specific cost of concurrency; batching (WorkManager, note 06) exists for this.

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Network/DB on main thread | `NetworkOnMainThreadException`, ANR | Coroutine on `Dispatchers.IO` |
| `Thread` outliving the Activity | Leak / crash | `lifecycleScope`/`viewModelScope` |
| Touching UI from a worker thread | `CalledFromWrongThreadException` | `withContext(Dispatchers.Main)` or post |
| `GlobalScope.launch` | Unstructured, uncancellable work | Use a lifecycle/viewmodel scope |
| `counter++` across coroutines | Lost updates (race) | `Mutex`/atomic/`StateFlow` |
| Catching `Exception` blindly and swallowing | Silent failures | Catch specific `IOException`/`HttpException` |
| No timeout on network calls | Spinner forever | OkHttp timeouts; `withTimeout` |

## 🧪 Practice Tasks
- [ ] **Task 1** — Write the same "download + show" feature twice: once with a raw `Thread`+`runOnUiThread`, once with `lifecycleScope.launch` + `withContext(Dispatchers.IO)`. Rotate the device mid-download; explain which one leaks/crashes and why.
- [ ] **Task 2** — Consume a public REST API with Retrofit + kotlinx.serialization, render results into a `RecyclerView` (note 03), and handle offline (`IOException`) by showing cached data from Room (note 04).
- [ ] **Task 3** (harder) — Reproduce a race condition: two coroutines incrementing a shared counter 100,000 times — first with a plain `var`, then with `Mutex`, then with `AtomicInteger`. Report the three final values and explain the differences.

## 📝 Research Journey
- **Why:** Async is the topic that separates "I can make a screen" from "I can make an app that doesn't freeze" — and it's the same problem the web solved with the event loop, so it bridges two of my tracks.
- **Struggle:** The `suspend` keyword was the hardest to internalize; I kept thinking it *creates* a thread. It doesn't — it *parks* the coroutine and frees the thread.
- **Aha moment:** Understanding that `AsyncTask`'s fatal flaw (unstructured lifecycle) is exactly what `lifecycleScope` fixes reframed "modern" APIs as *answers to specific accidents*, not arbitrary churn.
- **Career link:** concurrency correctness (races, cancellation, idempotency) is the operational core of any ML serving or data pipeline role I'm aiming at.

## 🔗 Related Topics
- [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md) — why the main thread matters (lifecycle, ANR)
- [`04-local-data-storage-and-access.md`](04-local-data-storage-and-access.md) — Room's off-main-thread contract & offline-first
- [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md) — cooperative concurrency in JS
- [`../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md) — JVM thread/collection semantics
- [`06-specialized-services-and-capstone.md`](06-specialized-services-and-capstone.md) — WorkManager, foreground services, background limits
- [SUGGESTED] `bridge-kotlin-flow-and-mvvm.md` — Flow/StateFlow as the correct way to expose async UI state

## 🤔 Open Questions
- [ ] Where is the line between "use a coroutine" and "use WorkManager"? (in-app async vs deferrable background work)
- [ ] How do we design cancellation so a half-finished network write doesn't corrupt local state?
- [ ] Can the offline-first sync protocol from note 04 be expressed purely with Flow operators, or does it need explicit conflict logic?

---
*A fast CPU with a blocked UI thread is still a frozen phone. Concurrency is about the frame budget, not the core count.*

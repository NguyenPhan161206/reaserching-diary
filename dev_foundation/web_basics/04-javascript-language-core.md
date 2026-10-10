# JavaScript Language Core — The Web Track's Language

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#JavaScript`, `#WebBasics`, `#LanguageCore`, `#Scoping`, `#Closures`, `#Async`, `#Modules`, `#DevTools`
**References:** MDN JavaScript Guide & Reference, ECMAScript 2024 Language Specification, *You Don't Know JS Yet* (Kyle Simpson), Node.js docs — "Modules: ECMAScript modules" and "Global objects"
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** Any one prior language (Java or C++) — see [`../../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md); conceptual HTML context in [`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md)

---

## 📌 Key Takeaways (TL;DR)
- **JavaScript is one language with two runtimes.** The *same* syntax runs in the **browser** (client-side scripting: `window`, `document`, no filesystem) and in **Node.js** (server-side scripting: `process`, `fs`, no DOM). Learning "JavaScript" means learning the language core, then learning what each host *adds* and *takes away*.
- **It is single-threaded but not blocking.** One call stack, one thing at a time — concurrency comes from the event loop parking I/O, not from threads. Everything in §7 exists because of this one sentence.
- **`===` is the operator you want; `==` is a coercion machine.** Almost every "JavaScript is weird" complaint is `==`, `+`, or truthiness hitting a type you did not expect.
- **`const` is the default; `let` is the exception; `var` is history.** `const`/`let` are block-scoped and hoisted-but-uninitialised (the temporal dead zone), which turns silent bugs into loud errors.
- **Functions are values and they carry their birth scope.** That single fact — *closures* — gives you counters, factories, module privacy, and every callback in the codebase you will read.
- **`this` is decided by how a function is *called*, except for arrow functions, where it is decided by where the function is *written*.**
- **Async is three generations of the same idea:** callbacks → Promises → `async`/`await`. `await` is not "pause the program"; it is "park this task and let the event loop run others". The ordering rules (microtasks before macrotasks) are the bridge to [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md).
- **Immutability is a habit, not a keyword.** `map`/`filter`/spread return *new* containers; `sort`/`push` mutate. Knowing which is which prevents the "my array changed by itself" bug.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 The web track's language (repo Section N)

This diary keeps exactly one language per track (AGENTS.md Section N). The **DSA track is Java + C++**. The **web track is JavaScript**. The **AI/ML track is Python**. This note is the web track's language note: it exists so that when you open the Joblink codebase (Next.js/React) or write a `fetch` handler in the browser, you can read the *logic* without re-learning syntax.

> Mental model: **Java and JavaScript share a name fragment and almost nothing else.** Both have C-style braces, `if`, `for`, and `class` — and that surface similarity is a trap. JavaScript has no static types at compile time, no integer/float split (`number` is IEEE-754 double, see §2), functions as first-class values, and no threads.

#### 1.2 One language, two runtimes

The spec defines the **language** (values, operators, functions). A **host environment** defines the **globals** and the **APIs**. There are two hosts that matter in this course:

```
        THE LANGUAGE (ECMAScript)  — identical in both
        values · operators · functions · objects · Promise · modules
                 │
        ┌────────┴─────────┐
        ▼                  ▼
  BROWSER (client-side)   NODE.js (server-side)
  window, document        process, global
  fetch, localStorage     fs, http, path
  ❌ no fs                ❌ no DOM
```

| | Browser (client-side scripting) | Node.js (server-side scripting) |
|---|---|---|
| Global object | `window` (= `globalThis`) | `global` (= `globalThis`) |
| DOM access | ✅ `document`, `window` | ❌ none (no HTML) |
| Filesystem | ❌ blocked (sandbox) | ✅ `fs`, `path` |
| Network | `fetch` to other origins, subject to CORS | `fetch`/`http`, no CORS |
| Who runs it | The user's machine, on page load/click | Your server, persistently |
| Trust | **Untrusted** — user can read/patch it | **Trusted** — never trust what the client sends |

**Why the distinction matters:** the *exact same* function `function add(a, b) { return a + b; }` runs in both. What differs is *what the code is allowed to touch*. Client code can be read and modified by anyone with DevTools; server code cannot. That asymmetry is the root of all web security (see [`../../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)) and the reason validation must happen on the server even if it "already" happened on the client.

#### 1.3 Single-threaded

There is **one** call stack. A JavaScript program runs one statement to completion before the next. It cannot be pre-empted by the OS scheduler the way a Java thread can. "Concurrency" is achieved by handing work to the host (timers, network, disk) and receiving the result as a **callback** later — that is the event loop (§7, and the deep dive in [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md)).

The practical consequence: **a long synchronous loop freezes the UI.** In Java you might spawn a thread; in the browser you must `await`, chunk, or move work to a Web Worker.

#### 1.4 Values: primitives vs objects

| Kind | Members | Copied how | Compared with `===` |
|---|---|---|---|
| **Primitive** | `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint` | by **value** (copied) | by value |
| **Object** | objects, arrays, functions, `Map`, `Set`, `Date`, … | by **reference** (the pointer is copied) | by reference (same memory?) |

```js
let a = { n: 1 };
let b = a;          // b points at the SAME object
b.n = 99;
console.log(a.n);   // 99  ← "changed by itself"

let x = 1;
let y = x;          // y is a copy
y = 2;
console.log(x);     // 1
```

**Call-by-value-of-reference:** when you pass an object to a function, the *reference* is passed by value. The function can mutate the object's contents, but it cannot make your variable point at a different object:

```js
function mutate(o) { o.n = 5; }
function reassign(o) { o = { n: 5 }; }  // only rebinds the local parameter

const obj = { n: 0 };
mutate(obj);    console.log(obj.n);  // 5
reassign(obj);  console.log(obj.n);  // still 5 (rebinding did nothing outside)
```

This is the single most common source of surprise for Java/C++ students: `const` does **not** make an object immutable — it only stops you reassigning the *binding*.

---

### 2. Types & Coercion

#### 2.1 `typeof` at a glance

| Expression | `typeof` | Note |
|---|---|---|
| `42`, `3.14`, `NaN`, `Infinity` | `"number"` | all IEEE-754 doubles |
| `10n` | `"bigint"` | arbitrary precision integer |
| `"hi"` | `"string"` | immutable |
| `true` | `"boolean"` | |
| `undefined` | `"undefined"` | "no value assigned" |
| `null` | `"object"` | **historical bug, unfixed for compatibility** |
| `Symbol()` | `"symbol"` | unique key |
| `{}`, `[]`, `/re/` | `"object"` | arrays and regexes are objects |
| `function(){}` | `"function"` | callable objects |

```js
typeof null          // "object"  ← the famous wart
Array.isArray([])    // true      ← use this, not typeof
null === null        // true      ← the only safe null check
```

**Reliable null check:** `x === null`. **Reliable array check:** `Array.isArray(x)`.

#### 2.2 The six falsy values

Every value is *truthy* except these **six**:

```js
false, 0, "", null, undefined, NaN        // ← the classic six
// variants: -0 and 0n also behave falsy in boolean context
```

Everything else is truthy — including `[]`, `{}`, `"0"`, `"false"`, and `-1`. This trips people constantly:

```js
if ("0") console.log("runs");   // runs! non-empty string is truthy
if ([])  console.log("runs");   // runs! empty array is truthy
if (-0)  console.log("no");     // does not run
```

#### 2.3 `==` vs `===` — the coercion trap

`===` (strict) compares type **and** value, no conversion. `==` (loose) *coerces* both sides to a common type using a labyrinthine algorithm.

```js
0  ==  ""        // true   (both → 0)
0  ==  false     // true
"" == false      // true   ← the chain: "" == 0 == false
null == undefined// true   (special-cased)
null == 0        // false  (null is only == to undefined)
NaN == NaN       // false  (NaN equals nothing)

"1" === 1        // false
"1" ==  1        // true
[] == false      // true   ([] → "" → 0)
```

**Rule for this course:** use `===` and `!==` everywhere. The only sanctioned `==` is `x == null`, which is true for both `null` and `undefined` — a compact "is absent" check.

#### 2.4 `NaN` and `Number.isNaN`

`NaN` ("Not a Number") is a `number` and, by IEEE-754, **not equal to itself**:

```js
NaN === NaN                 // false
Number.isNaN(NaN)           // true  ← correct test
isNaN("hello")              // true  ← WRONG: coerces "hello" → NaN first
Number.isNaN("hello")       // false ← correct: "hello" is a string, not NaN
```

Always prefer `Number.isNaN(x)` over the global `isNaN(x)`; the global coerces its argument and lies.

#### 2.5 Conversions and the dual `+`

Explicit conversion:

```js
Number("42")     // 42        Number("")     // 0
Number("42px")   // NaN       Number(null)   // 0
Number(undefined)// NaN       Number(true)   // 1
parseInt("08", 10)// 8        parseFloat("3.5x") // 3.5
String(42)       // "42"      Boolean("")    // false
(42).toString(2) // "101010"
```

The `+` operator is **addition if either side is a string, numeric addition otherwise** — it is overloaded:

```js
1 + 2            // 3
"1" + 2          // "12"   ← number stringified
1 + "2"          // "12"
1 + 2 + "3"      // "33"   (1+2=3 first, then 3+"3")
"1" + 2 + 3      // "123"  (left to right: "12" then "123")
[] + {}          // "[object Object]"
```

The `-`, `*`, `/` operators always coerce to **number**, so `"6" / "2" === 3`. Only `+` has the string branch. This asymmetry is the reason a "sum" that silently concatenates is the classic first bug.

> **Syllabus distinction box — client-side vs server-side scripting**
>
> The course syllabus names this under 3.1. The distinction is **not** about syntax; it is about *where the code executes and what it is trusted with*.
>
> - **Client-side scripting** = JavaScript running in the browser, delivered as part of the page (or a `<script>`/module). It can react to clicks, mutate the DOM, and call APIs. It **cannot** read local files or keep a secret. It is fast (no network round-trip) but **untrusted**.
> - **Server-side scripting** = JavaScript running in Node.js on the server (Express in `web_server/`). It can read the database, the filesystem, and environment secrets. It is **trusted** but must re-validate everything, because client checks are cosmetic.
> - **Same code, both runtimes:** a pure function like `formatPrice(cents)` runs unchanged in either. Code that touches `document` breaks in Node; code that touches `fs` breaks in the browser. The host is the boundary.
> - **How this course uses both:** browser JS lives in `web_basics/` (DOM, events, `fetch`); Node/Express lives in `web_server/` (routing, MVC, sessions, SSR). The bridge is AJAX/`fetch` — the browser asks, the server answers JSON.

---

### 3. Variables & Scope

| Keyword | Scope | Hoisted? | Reassignable? | Redeclarable? | Use when |
|---|---|---|---|---|---|
| `var` | function | yes (→ `undefined`) | yes | yes | never in new code |
| `let` | **block** `{}` | yes (TDZ, no access) | yes | no | a value that changes |
| `const` | **block** `{}` | yes (TDZ, no access) | no (binding fixed) | no | **default choice** |

**Hoisting** means declarations are lifted to the top of their scope at parse time, *before* execution.

- `var x` is hoisted and initialised to `undefined`. So `console.log(x)` *before* the declaration prints `undefined` (no error).
- `let`/`const` are hoisted but **uninitialised**: the span from scope entry to the declaration is the **Temporal Dead Zone (TDZ)**. Touching them there throws `ReferenceError`.

```js
console.log(v); // undefined   (var hoisted as undefined)
var v = 5;

console.log(l); // ❌ ReferenceError: Cannot access 'l' before initialization
let l = 5;
```

**Function declarations hoist fully** — name *and* body — so you may call them before their line. **Function expressions assigned to `var`/`let` do not**; only the variable is hoisted, and it is `undefined`/TDZ until the assignment runs:

```js
sayHi();        // ✅ "hi"  — declaration hoisted with body
function sayHi() { console.log("hi"); }

sayBye();       // ❌ TypeError: sayBye is not a function
var sayBye = function () { console.log("bye"); };
```

**Why `const` first:** it removes reassignment as a failure mode and documents intent. Use `let` only when the binding genuinely changes (loop counters, accumulators). Reach for `var` only when reading legacy code.

---

### 4. Functions & `this`

#### 4.1 Three ways to write a function

```js
// 1. Declaration — hoisted with its body; has its own `this` and `arguments`
function add(a, b) { return a + b; }

// 2. Expression — assigned to a variable; not usable before the assignment
const add2 = function (a, b) { return a + b; };

// 3. Arrow — concise; NO own `this`, NO `arguments`, cannot be a constructor
const add3 = (a, b) => a + b;      // implicit return when body is one expression
const add4 = (a, b) => { return a + b; }; // explicit return with braces
```

#### 4.2 `this` — dynamic vs lexical

For a **normal** function, `this` is decided **at call time** by *how* it is called:

| Call form | `this` is | Example |
|---|---|---|
| `obj.method()` | `obj` | method call |
| `fn()` (plain call) | `undefined` in strict/module, `globalThis` in sloppy mode | — |
| `new Fn()` | the freshly created object | constructor |
| `fn.call(o)` / `fn.apply(o)` | `o` (explicit) | borrow a method |
| `const g = obj.method; g()` | **lost** — becomes a plain call | the "lost `this`" bug |

An **arrow function** has **no own `this`**; it captures `this` **lexically** from where it was written:

```js
const counter = {
  n: 0,
  bad()  { setTimeout(function () { this.n++; }, 0); },  // this = global/undefined
  good() { setTimeout(() => { this.n++; }, 0); },        // this = counter ✅
};
```

This is why callbacks in modern code are almost always arrows: they do not tear `this` off the surrounding object.

#### 4.3 `call`, `apply`, `bind`

```js
function greet(greeting, punct) { return `${greeting}, ${this.name}${punct}`; }
const user = { name: "Ada" };

greet.call(user, "Hi", "!");    // Hi, Ada!   (args one by one)
greet.apply(user, ["Hi", "!"]) // Hi, Ada!   (args as an array)
const bound = greet.bind(user, "Hi"); // returns a NEW fn with `this` fixed
bound("?");                    // Hi, Ada?
```

`bind` is permanent: the bound `this` cannot be overridden by a later `.call`.

#### 4.4 Default params, rest, spread

```js
function order(item, qty = 1, ...extras) {   // `= 1` default; `...extras` rest array
  return { item, qty, extras };
}
order("tea");                 // { item:"tea", qty:1, extras:[] }
order("tea", 2, "sugar","ice")// { qty:2, extras:["sugar","ice"] }

const nums = [3, 1, 2];
Math.max(...nums);            // 3   (spread: array → argument list)
const copy = [...nums];       // shallow copy
```

#### 4.5 Closures — the real superpower

A **closure** is a function bundled with the *scope it was created in*. The inner function keeps its birth environment alive even after the outer function returns.

```js
function makeCounter() {
  let count = 0;                    // private — nobody outside can touch it
  return {
    inc: () => ++count,
    get: () => count,
  };
}
const c = makeCounter();
c.inc(); c.inc();
console.log(c.get());               // 2
console.log(typeof count);          // "undefined" — truly private
```

**Why it matters:** closures give you (a) private state without classes, (b) factories that remember configuration, and (c) every callback that captures a variable (`fetch(url).then(res => render(res, listId))`). The classic pitfall (`var` in a loop capturing one shared variable) is fixed by `let`, which creates a fresh binding per iteration.

#### 4.6 IIFE — the old module pattern

```js
const api = (function () {
  let secret = 0;                    // hidden
  return { bump: () => ++secret };
})();
api.bump();                          // 1
```

Before ES modules, an IIFE was the only way to get a private scope. You will see it in older codebases; know it, prefer modules (§6).

---

### 5. Collections

#### 5.1 Array methods

| Method | Returns | Mutates? | Use for |
|---|---|---|---|
| `map(fn)` | new array (same length) | no | transform each element |
| `filter(fn)` | new array (subset) | no | keep matching elements |
| `reduce(fn, init)` | one value | no | sum/group/build |
| `find(fn)` | first match or `undefined` | no | locate one item |
| `findIndex(fn)` | index or `-1` | no | locate a position |
| `forEach(fn)` | `undefined` | no | side effects only |
| `some(fn)` / `every(fn)` | boolean | no | existence / universal test |
| `sort(fn)` | the SAME array | **yes** | ordering (watch the comparator!) |

```js
const prices = [10, 25, 5];
prices.map(p => p * 2);        // [20, 50, 10]
prices.filter(p => p >= 10);   // [10, 25]
prices.reduce((s, p) => s + p, 0); // 40

// ⚠️ default sort is lexicographic on strings
[10, 25, 5].sort();            // [10, 25, 5]  (as text!)
[10, 25, 5].sort((a, b) => a - b); // [5, 10, 25] ✅
```

#### 5.2 Immutability habits

`map`/`filter`/`reduce`/spread return **new** containers; `push`/`pop`/`splice`/`sort`/`reverse` change the original. This matters in React/Next.js (Joblink), where mutating state silently breaks re-rendering. Prefer:

```js
const next = [...list, item];        // append without mutating
const updated = list.map(x => x.id === id ? { ...x, done: true } : x);
```

Note: `{ ...x }` and `[...list]` are **shallow** — nested objects are still shared.

#### 5.3 Destructuring & spread

```js
const { name, age = 0 } = user;                 // object destructure + default
const [first, ...rest] = [1, 2, 3];             // array destructure + rest
const merged = { ...defaults, ...overrides };   // shallow merge, later wins
```

#### 5.4 `Set` and `Map` — when they beat a plain object

| Need | Use | Why |
|---|---|---|
| unique values, fast membership | `Set` | `O(1)` `has`; dedupes automatically |
| keyed by **non-string** (object, number) | `Map` | object keys are coerced to strings |
| insertion-ordered iteration | `Map`/`Set` | both preserve insertion order |
| simple string-keyed record / JSON | plain object `{}` | serialises to JSON directly |

```js
const uniq = new Set([1, 1, 2, 3]);       // {1, 2, 3}
const counts = new Map();
counts.set("a", 1).set("b", 2);
counts.get("a");                          // 1
```

`Set`/`Map` are insertion-ordered; Java's `HashMap` is unordered. Keep that contrast in mind when porting DSA ideas into web code.

---

### 6. Modules

Before modules, every file shared the **global scope** — a collision waiting to happen. Modern code splits into **modules**, each with its own scope and an explicit `export`/`import` contract.

Two systems exist, and this is exactly the **browser/Node split** of §1.2:

| | ES Modules (ESM) | CommonJS (CJS) |
|---|---|---|
| Syntax | `import` / `export` | `require` / `module.exports` |
| Host | **Browser** (and modern Node) | **Legacy Node** |
| Loaded | statically, before execution | at call time |
| Default in Node | with `"type": "module"` | default without it |
| Top-level `await` | ✅ | ❌ |

```js
// ES Modules
export const PI = 3.14;
export function area(r) { return PI * r * r; }
export default function () { /* … */ }

import area, { PI } from "./math.js";
```

```js
// CommonJS
const { PI } = require("./math");
module.exports = { area };
```

To make Node treat `.js` files as ESM, set the package manifest:

```json
{ "type": "module" }
```

**Why it matters:** the Joblink codebase is ESM-style throughout. When you see `import { useState } from "react"`, the bundler resolves and links modules *before* the code runs — which is why imports are hoisted and why a typo in a module path fails at load, not mid-function.

---

### 7. Async Foundation (bridge to Note 05)

#### 7.1 Three generations of the same idea

**Generation 1 — callbacks.** Pass a function to run later. Works, but nesting produces "callback hell" and error handling is ad hoc.

```js
getUser(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => { /* … */ });
});
```

**Generation 2 — Promises.** A value that will exist later, with a chainable interface and centralised errors.

```js
getUser(id)
  .then(user => getOrders(user.id))
  .then(orders => render(orders))
  .catch(err => handle(err));
```

**Generation 3 — `async`/`await`.** Async code that reads sequentially. `async` makes a function return a Promise; `await` unwraps one.

```js
async function showOrders(id) {
  try {
    const user   = await getUser(id);
    const orders = await getOrders(user.id);
    render(orders);
  } catch (err) {           // one place to catch every await above
    handle(err);
  }
}
```

#### 7.2 Error handling

A rejected Promise that nobody handles triggers an "unhandled rejection" warning. `try/catch` catches `await`ed rejections; `.catch` catches chain rejections. For parallel work, `Promise.all` rejects fast, `Promise.allSettled` waits for all and reports each outcome.

```js
const [a, b] = await Promise.all([fetchA(), fetchB()]); // both start together
```

#### 7.3 Microtask vs macrotask — preview

`await`/`.then` callbacks go on the **microtask** queue; `setTimeout`/I/O go on the **macrotask** queue. The event loop **drains all microtasks before the next macrotask**:

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1, 4, 3, 2
```

Read it as: synchronous first (`1`, `4`), then microtasks (`3`), then macrotasks (`2`). This ordering is the whole subject of [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md) — and it is why `setTimeout(0)` is *not* immediate.

---

## 🧾 Command / Code Cheat Sheet

| Construct | What it does | When to use | Example |
|---|---|---|---|
| `typeof x` | returns a string type tag | quick type check (`null` lies) | `typeof [] === "object"` |
| `x === y` / `x !== y` | strict compare, no coercion | **always** | `"1" === 1 // false` |
| `const` / `let` | block-scoped binding | `const` default, `let` if reassigned | `const pi = 3.14` |
| `(...args) => …` | arrow fn, lexical `this` | callbacks inside objects | `list.map(x => x * 2)` |
| `arr.map/filter/reduce` | transform / select / fold, no mutation | everyday array work | `[1,2].reduce((a,b)=>a+b,0)` |
| `{...o}` / `[...a]` | shallow clone / merge | immutable updates | `{...state, done:true}` |
| `import` / `export` | ES module link | all modern web code | `import { useState } from "react"` |
| `Promise.resolve().then(fn)` | schedule a microtask | async ordering | `Promise.resolve().then(run)` |
| `async` / `await` | write async sequentially | any I/O call | `const r = await fetch(u)` |
| `Number.isNaN(x)` | true NaN test | validating numeric input | `Number.isNaN(NaN)` |

---

## 🧪 Practical Lab

> Run each with `node lab.js`. Compare stdout to the expected block before reading on.

**Lab 1 — Coercion & truthiness**

```js
// lab1.js
console.log(1 + 2 + "3");        // ?
console.log("1" + 2 + 3);        // ?
console.log([] == false);        // ?
console.log(null == undefined);  // ?
console.log(NaN === NaN);        // ?
console.log(Boolean("0"));       // ?
```
```text
Expected stdout:
33
123
true
true
false
true
```
*What to notice:* `+` goes left to right and switches to concatenation the moment a string appears; `"0"` is truthy because it is non-empty.

**Lab 2 — Closures**

```js
// lab2.js
function makeAccumulator(start) {
  let total = start;
  return (n) => (total += n);
}
const acc = makeAccumulator(100);
console.log(acc(10), acc(5));
console.log(typeof total);   // out of scope
```
```text
Expected stdout:
110 115
undefined
```
*What to notice:* `total` survives between calls (closure) yet is invisible outside.

**Lab 3 — Async ordering preview**

```js
// lab3.js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```
```text
Expected stdout:
A
D
C
B
```
*What to notice:* sync (`A`, `D`) → microtask (`C`) → macrotask (`B`). Full mechanism in Note 05.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---|---|---|
| `typeof null === "object"` | null passes object checks | use `x === null`; `Array.isArray` for arrays |
| `==` coercion | `0 == ""`, `[] == false` are `true` | use `===` / `!==` |
| `NaN` comparisons | `x === NaN` never true | `Number.isNaN(x)` |
| Arrow `this` | method callback sees wrong `this` | normal method or `.bind`, or arrow *outside* |
| Shallow copy | editing a "copied" nested object changes the original | deep copy / spread each level |
| Forgetting `return` in a block-bodied arrow | function returns `undefined` | `x => { return y; }` |
| `parseInt("08")` | old engines read it as octal → `0` | `parseInt(s, 10)` or `Number(s)` |
| `let` in closures | loop variable shared, callbacks all see last value | `let` (per-iteration binding) or IIFE |
| `sort()` without comparator | `[10,5,1]` sorts as text | `sort((a,b) => a-b)` |
| Missing `await` | you get a `Promise`, not the value | `await` it, or `.then` |

---

## 🔗 Tech Cross-References

- **DOM & Browser** — JavaScript is the *only* language the browser runs; the DOM is the object tree it mutates. The event loop that schedules `await`/`setTimeout` is the subject of [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md).
- **Node.js / Server-side (Express)** — the *same language*, second runtime. Server-side scripting runs in Node with `fs`/`process` and no DOM: [`../../web_server/01-static-dynamic-ssr-vs-csr.md`](../web_server/01-static-dynamic-ssr-vs-csr.md).
- **Databases (PostgREST)** — the server returns JSON; JS turns it into objects (`JSON.parse`) and renders it. Every list UI you will build consumes rows this layer produces: [`../../databases/README.md`](../databases/README.md).
- **Java/C++ contrast (DSA track)** — types, `==` vs `.equals`, integer/float split, threads vs the event loop; the contrast sharpens both: [`../../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md).

## 🧪 Practice Tasks

> No solutions. Do them, then ask the AI to review your reasoning and edge cases.

- [ ] **Task 1 — Coercion detective.** Without running it, predict every line of: `console.log("5" - 2, "5" + 2, [] + [], [1] + 1, NaN || "fallback")`, then verify with `node -e`. Explain each result by the coercion rules.
- [ ] **Task 2 — Private counter.** Write `makeBank(balance)` returning `{ deposit, withdraw, statement }` where `balance` cannot be read or written except through those methods. Prove privacy by trying (and failing) to read `balance` from outside.
- [ ] **Task 3 — Immutable pipeline.** Given `const items = [{id:1, qty:2, price:10}, …]`, produce a *new* array of `{id, total}` for items with `qty > 0`, sorted by `total` descending, using only non-mutating methods. Then verify the original array is untouched.
- [ ] **Task 4 — Async ordering.** Predict the console order of a script mixing `console.log`, `queueMicrotask`, `Promise.resolve().then`, and two `setTimeout`s (0ms and 10ms). Run it, then explain the result with the microtask/macrotask rule from Note 05.

## 📝 Research Journey

- **Why:** The Web Application Development course is my first serious contact with JavaScript, and it is the *one* language of the web track (AGENTS.md Section N). I know Java and C++, so I do not need "programming" — I need the *differences*: dynamic types, closures, `this`, and the single-threaded event loop.
- **Struggle:** `this` was the hardest. Coming from Java, I assumed it behaved like `self` in a method. The mental shift — *`this` is a property of the call site, not the function* — took several broken callbacks to internalise, and arrow functions finally made it click.
- **Aha moment:** Realising `await` does not stop the world. It parks one task and lets the event loop run others; the "1, 4, 3, 2" output in §7.3 was the moment async stopped being magic.
- **Career link:** As an AI Engineer in Economics/Business, my interface to a model is almost always a client app — dashboards, chat UIs, analytics pages — all JavaScript. Models run in Python on the server; the *product* is JS in the browser. Reading and writing that layer is what turns a notebook into something a business can use.

## 🔗 Related Topics

- [`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md) — the markup this language mutates.
- [`08-forms-fetch-and-multipart-upload.md`](08-forms-fetch-and-multipart-upload.md) — where JS collects user input and sends it to the server.
- [`../../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md) — the server-side counterpart that answers the requests JS sends.
- [SUGGESTED] `09-error-handling-and-debugging.md` — because `try/catch`, DevTools, and `console` methods deserve their own note; errors are how you actually learn the runtime.
- [SUGGESTED] `10-json-and-data-shapes.md` — JSON is the lingua franca between browser JS and the server/PostgREST; a dedicated note would connect this layer to `databases/`.

## 🤔 Open Questions

- [ ] How exactly does the microtask queue interact with the rendering opportunity in a browser — does a `Promise.then` always run before the next paint?
- [ ] When should I reach for a Web Worker instead of chunking a long loop, and what is the data-transfer cost (structured clone vs `SharedArrayBuffer`)?
- [ ] Does `async`/`await` have a performance cost versus hand-written `.then` chains, or is the sugar free today?

---
*One language, two runtimes. Learn the language, then learn what each host adds and denies.*

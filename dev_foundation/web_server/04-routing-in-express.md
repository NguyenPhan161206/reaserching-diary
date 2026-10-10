# Routing & Middleware in Express — How a URL Becomes a Response

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Express`, `#Routing`, `#Middleware`, `#NodeJS`, `#Router`, `#ErrorHandling`, `#DevTools`
**References:** Express.js 4 guide — "Routing", "Writing middleware", "Error handling", "Express Router"; Express 5 migration notes — async error handling; MDN — "HTTP request methods", "HTTP response status codes"; Node.js docs — `process.env`, `EventEmitter`
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) (Express, `req`/`res`, ports), [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (functions, callbacks, async/await), [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md) (controller = route handler)

---

## 📌 Key Takeaways (TL;DR)
- **Routing = matching an incoming `(method, path)` against an ORDERED list of middleware stacks.** The browser only ever says *"GET /posts/42"*; Express walks its list and hands the request to the **first** handler that responds.
- **Order is logic, not decoration.** `app.use`/`app.get` run in the exact order you register them. A catch-all registered too early swallows every route below it.
- Every Express app is **a function that runs handlers in sequence** until one of them *responds* (`res.send`, `res.json`, `res.render`, `res.redirect`) or errors (`next(err)`).
- `req.params` are always **strings**; `req.body` is empty unless you mount `express.urlencoded()`/`express.json()`; `req.query` comes from the query string.
- An **error handler is just middleware with four arguments** `(err, req, res, next)` — and in Express 4, a thrown error inside an `async` handler is *silently lost* unless you `try/catch` and call `next(err)`.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

Hold one picture before typing any route: **an Express app is an ordered array of functions**, and a request is an item passed down that array.

```
GET /posts/42   ──┐
                  ▼
 ┌─────────────────────────────────────────────────────────────────┐
 │  app = [ fn1, fn2, fn3, fn4, errorHandler ]   (registration order)│
 │                                                                   │
 │  request ──► fn1 ──next()──► fn2 ──next()──► fn3 ──res.send()──►  │
 │               │                │               │                  │
 │            (logging)      (express.json)   (route handler)        │
 │                                            RESPONDS → chain stops │
 │                                                                   │
 │  if fn calls next(err) ───────────────────────────► errorHandler  │
 └─────────────────────────────────────────────────────────────────┘
```

Each function receives `(req, res, next)` and makes exactly one of three choices:

1. **Respond** — `res.send()`, `res.json()`, `res.render()`, `res.redirect()`. The chain **stops here**; the browser gets its answer.
2. **Continue** — call `next()` to hand control to the *next* function in the list.
3. **Fail** — call `next(err)` (or throw) to jump to the error handler, skipping all normal handlers.

**Why this mental model matters:** almost every confusing Express bug is an *ordering* or *controls* bug, not a syntax bug. "My auth middleware doesn't run" means it is registered *after* the route. "The request hangs forever" means a handler neither responded nor called `next()`. "My 500 has no message" means nobody forwarded the error. Once you see the app as a sequential list, these stop being mysteries and become positions in the array.

```
   Two rules to memorise:
   ───────────────────────
   • A handler that neither responds nor calls next()  →  request HANGS
   • Order of registration === order of execution
```

### 2. Path & Method Matching

Express exposes one method per HTTP verb:

```js
app.get(path, handler);     // read
app.post(path, handler);    // create
app.put(path, handler);     // replace
app.patch(path, handler);   // partial update
app.delete(path, handler);  // remove
app.use(path, handler);     // ALL methods + prefix match (see §3)
```

**Path forms, from simple to dynamic:**

| Form | Example | Matches | Access |
|---|---|---|---|
| Static | `/about` | only `/about` | — |
| Route param | `/posts/:id` | `/posts/42`, `/posts/hello` | `req.params.id` |
| Multiple params | `/users/:uid/posts/:pid` | `/users/7/posts/3` | `req.params.uid`, `req.params.pid` |
| Resource (REST) | `/posts/:id/comments` | nested resource | `req.params.id` |

```js
app.get("/posts/:id", (req, res) => {
  console.log(typeof req.params.id); // "string" — ALWAYS a string, even for "42"
  res.json({ id: req.params.id });
});
```

**Route params are strings.** `GET /posts/42` gives `req.params.id === "42"`, not `42`. Passing that straight into a database where the column is an integer may work by coercion, but comparing or arithmetic on it will bite you. Convert deliberately: `const id = Number(req.params.id);` — and guard against `NaN`. (A later note adds validation middleware; for now, never assume a number.)

**Query strings** are the `?key=value&…` part, accessed via `req.query` and always strings too:

```js
// GET /posts?page=2&limit=10&tag=ai
app.get("/posts", (req, res) => {
  const page = Number(req.query.page) || 1;
  const limit = Math.min(Number(req.query.limit) || 10, 100); // clamp!
  res.json({ page, limit, tag: req.query.tag });
});
```

**Body** (`req.body`) does **not exist** until a body-parsing middleware runs. For an HTML form it is `express.urlencoded()`; for fetch/JSON it is `express.json()`:

```js
app.use(express.urlencoded({ extended: true })); // <form> posts → req.body
app.use(express.json());                          // fetch JSON    → req.body

app.post("/posts", (req, res) => {
  res.status(201).json({ received: req.body });
});
```

**Conventions worth adopting now.** Model a resource with the REST shape — `GET /posts` (list), `GET /posts/:id` (one), `POST /posts` (create), `PATCH /posts/:id` (update), `DELETE /posts/:id` (remove). Consistent URLs are not just tidy; they make the routing layer predictable and map one-to-one onto the controller actions from [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md).

### 3. Middleware — The `(req, res, next)` Triple

Middleware is any function with the signature `(req, res, next)` that runs *between* the request arriving and the response being sent. Two ways to attach it:

- `app.use(fn)` — runs for **every** request whose path matches the mount point (default `/`).
- `app.METHOD(path, fn)` — runs only when **method + path** match.

```js
// custom logging middleware — runs for every request
function logger(req, res, next) {
  const start = Date.now();
  res.on("finish", () => {           // fires after the response is sent
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${Date.now() - start}ms`);
  });
  next();                            // MUST continue the chain
}

app.use(logger);
```

**Built-in middleware you will use constantly:**

| Middleware | What it does | Mount when |
|---|---|---|
| `express.json()` | parse `application/json` body → `req.body` | any API receiving JSON |
| `express.urlencoded({ extended: true })` | parse HTML form body → `req.body` | classic SSR forms |
| `express.static("public")` | serve files from a folder | images, CSS, client JS |

**A custom auth guard** — the pattern behind [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md):

```js
function requireAuth(req, res, next) {
  if (req.session && req.session.userId) return next();     // allowed → continue
  return res.redirect("/login");                            // not allowed → respond
}
app.get("/dashboard", requireAuth, (req, res) => res.render("dashboard"));
```

**Error-first middleware (four args).** Express recognises an error handler by its **arity**: exactly four parameters. It must be registered **last**, after all routes:

```js
app.use((err, req, res, next) => {   // 4 args => Express treats this as the error handler
  console.error(err);
  res.status(err.status || 500).json({ error: err.message || "Server Error" });
});
```

In the chain, `next(err)` (or a synchronous `throw`) skips every normal handler and lands here.

**Async handlers and the Express 4 trap.** Express 4 does **not** catch a rejected promise from an `async` handler. The request simply hangs, and no error handler runs:

```js
// ❌ Express 4: if findById rejects, this is an UNHANDLED rejection — request hangs
app.get("/posts/:id", async (req, res) => {
  const post = await Post.findById(req.params.id);
  res.json(post);
});

// ✅ wrap in try/catch and forward to the error handler
app.get("/posts/:id", async (req, res, next) => {
  try {
    const post = await Post.findById(req.params.id);
    if (!post) return res.status(404).json({ error: "Not found" });
    res.json(post);
  } catch (err) {
    next(err);                       // hands the failure to the error handler
  }
});
```

To avoid repeating `try/catch`, wrap the handler once:

```js
// a tiny async wrapper (this is what express-async-errors / express 5 give you for free)
const wrap = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get("/posts/:id", wrap(async (req, res) => { /* may throw freely */ }));
```

> **Express 5 note:** Express 5 forwards rejected promises from async handlers to the error handler automatically, so the manual `wrap` becomes optional. Know which major version you are on before deleting the guard.

### 4. `express.Router` — Modularizing Routes

A `Router` is a **mini-app**: it holds its own middleware stack and paths, and gets *mounted* onto a path on the main app. This is how a real project avoids a 500-line `server.js`.

```js
// routes/posts.js
const express = require("express");
const router = express.Router();
const postsController = require("../controllers/postsController");
const { requireAuth } = require("../middlewares/auth");

router.get("/", postsController.index);           // GET  /posts
router.get("/new", requireAuth, postsController.newForm); // GET /posts/new  (BEFORE /:id!)
router.get("/:id", postsController.show);         // GET  /posts/:id
router.post("/", requireAuth, postsController.create);    // POST /posts

module.exports = router;
```

```js
// server.js — mount the router under a prefix
const postsRouter = require("./routes/posts");
app.use("/posts", postsRouter);                   // every route inside is now prefixed with /posts
```

**The mount rule:** inside the router, paths are **relative** (`/`, `/:id`); the prefix comes from `app.use("/posts", …)`. So `router.get("/:id")` becomes `GET /posts/:id`.

**When to mount a router:** the moment one resource has more than a couple of routes, or two people are editing the same routes file. One router per resource (`posts`, `users`, `comments`) keeps files small and merges clean.

**The "controllers export handlers" pattern.** The router should contain *no* logic — only wiring. The handlers live in controllers (see [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md)), so the router reads like a table of contents:

```
server.js  ──mounts──►  routes/posts.js  ──points to──►  controllers/postsController.js
   (config)                (URL wiring)                      (req/res logic)
```

### 5. 404 and Centralized Errors

Two catch-alls, registered **after all routes**, handle everything the routes did not:

```js
// 1. 404 — reached only if NO route responded above
app.use((req, res, next) => {
  res.status(404).render("errors/404", { url: req.originalUrl });
  // or: res.status(404).json({ error: "Not found" });
});

// 2. error handler — MUST have 4 args and be LAST
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).render("errors/500", { message: err.message });
});
```

Because these are registered last, they only see requests that fell through everything above — which is exactly what "not found" means in an ordered stack.

**Status codes you will actually use, and when:**

| Code | Name | When to use |
|---|---|---|
| `200` | OK | normal successful GET / update |
| `201` | Created | after a successful POST that made a resource |
| `204` | No Content | DELETE succeeded, nothing to return |
| `301` | Moved Permanently | URL changed forever (SEO) |
| `302` | Found (temp redirect) | post-login redirect, `res.redirect("/")` |
| `400` | Bad Request | malformed/invalid input from the client |
| `401` | Unauthorized | **not logged in** (authentication) |
| `403` | Forbidden | logged in, but **not allowed** (authorization) |
| `404` | Not Found | route/resource does not exist |
| `500` | Internal Server Error | your bug — never expose the stack in production |

`res.redirect("/login")` sends a `302`; `res.redirect(301, "/new-home")` sends a permanent one. `401` vs `403` is the classic exam trap: **401 = "who are you?" (not authenticated); 403 = "I know who you are, still no" (not authorized).**

### 6. Pitfalls Solved (the ones this note is about)

- **Order kills routes.** `app.use((req,res)=>res.status(404)…)` placed *before* the real routes makes every request a 404 — the catch-all is simply *earlier* in the array. Always register catch-alls last.
- **`/posts/:id` capturing `"new"`.** If `GET /posts/:id` is registered before `GET /posts/new`, then `GET /posts/new` matches `:id` with `id === "new"`. Register the **specific literal route first**, the parameterised one after.
- **Numbers that are strings.** `req.params.id` and `req.query.page` are strings; convert and validate before arithmetic or DB comparisons.
- **Case sensitivity & trailing slash.** By default Express routing is case-insensitive (`/About` matches `/about`) and a trailing slash is *not* required. These are **defaults you may not want**; `app.set("case sensitive routing", true)` and `app.set("strict routing", true)` make them explicit — pick a house style and stick to it so links are predictable.

---

## 🧾 Command / Code Cheat Sheet

| Command / Code | What it does | When to use | Example / Output |
|---|---|---|---|
| `app.get(path, fn)` | Register a GET route | Read endpoints / pages | `GET /posts` → handler |
| `app.post(path, fn)` | Register a POST route | Form submits, creates | `POST /posts` → `201` |
| `app.put/patch/delete(path, fn)` | Full/partial update, delete | REST resource edits | `PATCH /posts/:id` |
| `app.use([path], fn)` | Mount middleware (all methods, prefix match) | Logging, parsers, static, routers | runs for every request below it |
| `req.params` | Path variables from `:name` | Reading `/posts/:id` | `{ id: "42" }` (string!) |
| `req.query` | Parsed query string | Reading `?page=2` | `{ page: "2" }` |
| `req.body` | Parsed body (**needs parser middleware**) | Form/JSON input | `{ title: "Hi" }` or `{}` if unparsed |
| `express.json()` / `express.urlencoded()` | Body parsers | Mount before POST routes | populates `req.body` |
| `express.Router()` | Modular route handler | Split routes per resource | `app.use("/posts", router)` |
| `next()` | Continue to the next middleware | End of a pass-through middleware | chain advances |
| `next(err)` | Jump to the error handler | In `catch` blocks | 4-arg handler runs |
| `app.use((err,req,res,next)=>{})` | Centralized error handler | Registered LAST | uniform `500` response |
| `res.status(n).json(...)` | Set status + send JSON | APIs, error responses | `res.status(404).json({error})` |
| `res.redirect([301,] url)` | Send a redirect (`302` default) | After login / moved URLs | `res.redirect("/login")` |
| `curl -i URL` | Show status + headers + body | Verify any route quickly | `HTTP/1.1 200 OK …` |

---

## 🧪 Practical Lab

> Two labs, fully runnable, each with the expected `curl -i` output to verify against.

**Lab 1 — one file exercising params, query, and body**

```bash
mkdir /tmp/routing-lab && cd /tmp/routing-lab
npm init -y
npm i express
```

```js
// server.js
const express = require("express");
const app = express();

app.use(express.json());                              // so POST JSON reaches req.body
app.use(express.urlencoded({ extended: true }));      // so HTML forms reach req.body

app.get("/", (req, res) => res.send("Home"));

// :id arrives as a STRING — convert deliberately
app.get("/posts/:id", (req, res) => {
  res.json({ id: req.params.id, type: typeof req.params.id });
});

// query strings are strings too; clamp limit to avoid abuse
app.get("/search", (req, res) => {
  const limit = Math.min(Number(req.query.limit) || 10, 50);
  res.json({ q: req.query.q, limit });
});

// body only exists because the parsers are mounted above
app.post("/posts", (req, res) => {
  res.status(201).json({ created: true, got: req.body });
});

app.listen(3000, () => console.log("listening on http://localhost:3000"));
```

Run it, then in a **second terminal**:

```bash
curl -i http://localhost:3000/
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=utf-8
#
# Home

curl -i http://localhost:3000/posts/42
# HTTP/1.1 200 OK
# Content-Type: application/json; charset=utf-8
#
# {"id":"42","type":"string"}          ← note: STRING, not number

curl -i "http://localhost:3000/search?q=ai&limit=999"
# HTTP/1.1 200 OK
#
# {"q":"ai","limit":50}                ← clamped from 999 to 50

curl -i -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Hello"}'
# HTTP/1.1 201 Created
#
# {"created":true,"got":{"title":"Hello"}}
```

*What to notice:* `req.params.id` is a **string**; `req.query.limit` is a **string** and needs clamping; `req.body` is populated **only** because `express.json()`/`express.urlencoded()` are mounted. Remove the parsers and the POST body becomes `{}` — the classic missing-parser bug.

**Lab 2 — 404 for unknown paths + a uniform error response**

```js
// errors.js — same routes as Lab 1, plus catch-alls
const express = require("express");
const app = express();
app.use(express.json());

app.get("/posts/:id", (req, res, next) => {
  try {
    if (req.params.id === "0") throw new Error("simulated failure"); // a 500 path
    res.json({ id: req.params.id });
  } catch (err) {
    next(err);                              // forward to the error handler
  }
});

// 404 — only reached if no route above responded
app.use((req, res) => {
  res.status(404).json({ error: "Not found", path: req.originalUrl });
});

// error handler — 4 args, registered LAST
app.use((err, req, res, next) => {
  console.error(err.message);
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log("listening on http://localhost:3000"));
```

```bash
curl -i http://localhost:3000/posts/42
# HTTP/1.1 200 OK
# {"id":"42"}

curl -i http://localhost:3000/does-not-exist
# HTTP/1.1 404 Not Found
# {"error":"Not found","path":"/does-not-exist"}

curl -i http://localhost:3000/posts/0
# HTTP/1.1 500 Internal Server Error
# {"error":"simulated failure"}
```

*What to notice:* the unknown path hit the **404 catch-all** because no route responded; the thrown error skipped the route and landed in the **500 handler** because `next(err)` forwarded it. Move the 404 middleware *above* `/posts/:id` and the 200 route dies — proving that **order is logic.**

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---|---|---|---|
| Middleware registered after the route | Auth/logging "doesn't run" | The chain had already responded above it | Register middleware **before** the routes it protects |
| Forgetting `next()` | Request hangs, no response, no error | A middleware neither responded nor continued | Call `next()` at the end of pass-through middleware |
| Missing body parser | `req.body` is `{}` on POST | No parser ran before the handler | `app.use(express.json())` / `express.urlencoded()` |
| Async throw swallowed | Request hangs; error handler never runs | Express 4 does not catch async rejections | `try/catch` + `next(err)`, or the `wrap` helper (Express 5 auto-forwards) |
| `/:id` before `/new` | `/posts/new` treated as a post with `id="new"` | First match wins; `:id` matches any segment | Put literal routes before parameterised ones |
| Params treated as numbers | NaN / wrong DB comparison | `req.params`/`req.query` are always strings | `Number(...)` + validate, or parse in middleware |
| Catch-all too early | Every route returns 404 | The 404 handler is earlier in the array | Register catch-alls **last** |
| Port already in use | `Error: listen EADDRINUSE :::3000` | A stale server owns the port | `lsof -i :3000` then `kill <pid>` — see [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) |
| Secrets/stack leaked in `500` | Stack trace in the browser | `res.send(err.stack)` shipped to clients | Log the stack, return a generic message + status |
| `path-to-regexp` error on `"*"` | `TypeError: Missing parameter name` (Express 5) | Wildcard syntax changed in Express 5 | Use a catch-all middleware `app.use((req,res)=>…)` instead of `app.get("*")` |

---

## 🔗 Tech Cross-References
- **[MVC architecture]** — the route handler *is* the controller from note 03; routers wire URLs to controllers that stay thin. See [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md).
- **[Authentication]** — login guards are middleware; `req.session.userId` checked in `requireAuth` is the seam where sessions meet routing. See [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md).
- **[Databases / Knex]** — controllers reached by these routes are the ones that call the data layer and await rows. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **[Git & GitHub]** — routers and controllers are tracked, versioned files; a change to one route is one reviewable diff. See [`../../git_github/README.md`](../../git_github/README.md).

## 🧪 Practice Tasks
> No solutions provided. Do them, then ask the AI to grade your evidence (the `curl -i` output and the code).

- [ ] **Task 1 — Param vs query.** Build `GET /users/:username` that returns a small profile, plus a `?full=1` query flag that adds extra fields. Capture `curl -i` for both, and prove (with `typeof`) that the param and the flag arrive as strings.
- [ ] **Task 2 — Order experiment.** Register `GET /posts/:id` **before** `GET /posts/new`, then request `/posts/new`. Record what happens, swap the two lines, and record it again. Write one sentence stating the rule.
- [ ] **Task 3 — Guard as middleware.** Write a `requireAuth` middleware that redirects to `/login` when a fake `req.session.userId` is missing, and protects `/dashboard`. Prove the unauthenticated response is a `302` to `/login`, then simulate a login and show it reaching the dashboard.
- [ ] **Task 4 (harder) — Router + error handler.** Split `posts` and `users` into two `express.Router()` files mounted at `/posts` and `/users`, add a 404 catch-all and a centralized error handler, and demonstrate with `curl -i` a `200`, a `404`, and a forced `500` (an async route that throws).

## 📝 Research Journey
- **Why:** I could write `app.get` calls, but I had no real model of *why* "middleware order matters" beyond a warning in tutorials. My routes either worked by luck or broke mysteriously, and the fix was always "move this line up" without me understanding the mechanism.
- **Struggle:** For a while an error handler and a normal middleware looked identical — both were "a function with req/res". The four-argument signature felt like an arbitrary Express quirk rather than the single mechanism that decides who catches an error. My biggest real bug was an `async` handler that threw: nothing happened at all — no 500, no log, just a request that never ended. I blamed the browser before I understood Express 4 does not catch promises.
- **Aha moment:** Drawing the app as an **ordered array of functions** and tracing one request through it by hand. The moment I saw `next()` as "go to the next array slot" and `next(err)` as "skip ahead to the four-arg slot", ordering stopped being a superstition. The second click was realising 404 is not special — it is just the first handler that eventually responds when nothing above did.
- **Career link:** Every service I ship as an AI Engineer will sit behind a router and a middleware chain — an auth guard in front of an inference route, a logging/tracing middleware, a uniform error handler that hides stack traces. The habits here (order is logic, forward your errors, never leak internals) are the same ones that keep a model endpoint safe and debuggable in production.

## 🔗 Related Topics
- [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) — what `res.render` hands off to once a route responds.
- [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md) — how the browser produces the `req.body` your POST routes parse.
- [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md) — the controller that every route points at.
- [SUGGESTED] `09-routing-validation-and-param-middleware.md` — `express-validator`-style param/body validation as middleware. *Important because:* this note converts params manually and clamps by hand; a validation middleware is the production-grade next step.
- [SUGGESTED] `bridge-routing-to-nextjs-route-handlers.md` — maps this ordered middleware model onto Next.js route handlers and middleware. *Important because:* it connects what you learn here to the Joblink (Next.js) stack.

## 🤔 Open Questions
- [ ] With Express 5 forwarding async rejections automatically, is the `try/catch` + `next(err)` pattern now redundant, or does it still buy clearer domain-level error messages?
- [ ] Where should `res.status(404)` for a *missing resource* (found route, absent row) live — the controller, a service, or a shared helper — without duplicating it everywhere?
- [ ] How does the router's prefix mounting interact with middleware that must run *only* for one router (`router.use` vs `app.use`)?
- [ ] Is there a cost to deep middleware chains on every request, and how would you measure it?

---
*The app is an ordered list. `next()` walks forward; a response stops the walk; `next(err)` jumps to the last handler. Order is logic.*

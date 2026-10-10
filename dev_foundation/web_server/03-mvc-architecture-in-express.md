# MVC Architecture in Express — Separating Data, Presentation, and Orchestration

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#MVC`, `#Express`, `#SSR`, `#Architecture`, `#SeparationOfConcerns`, `#Knex`, `#NodeJS`, `#Handlebars`
**References:** Trygve Reenskaug, *Models–Views–Controllers* (Xerox PARC, 1979); Express.js guide — "Routing", "Using middleware"; Knex.js docs — "Query Builder", "Migrations"; Handlebars.js docs — "Built-in helpers"; MDN — "MVC"
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md) (what SSR produces), [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) (Express, `req`/`res`, `app.listen`), [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (JS functions, modules, async/await)

---

## 📌 Key Takeaways (TL;DR)
- **MVC is nothing more than a partition of responsibilities**: the **Model** owns data and the rules about it, the **View** owns presentation, the **Controller** owns the *glue* between them (read request → ask Model → hand data to View).
- In an Express SSR app the **route handler IS the controller**. MVC is not a framework feature — it is a discipline you choose, enforced mainly by **what you are allowed to write where**.
- The single most important rule in this course: **no SQL in the controller, no HTML in the controller, and views never touch the database.** Each layer talks only to its neighbour.
- The payoff is *testability and change-resistance*: you can restyle every page without touching a query, and test business logic without a browser.
- The opposite of MVC — the **fat controller** — starts innocently and ends as an untestable file where the header, the SQL, the JSON, and the validation are all tangled together.

---

## 🧠 Detailed Notes

### 1. Core Concepts — What MVC Actually Is

MVC was described by **Trygve Reenskaug at Xerox PARC in 1979**, in the era of Smalltalk-80, to solve a very concrete problem: a user interface has **state** (a document, a list, a selection), **display** (how it is drawn on screen), and **input** (what the user does). Those three concerns change for *different reasons* and at *different times*. Coupling them means every UI tweak risks breaking the data logic. MVC is the response to that pressure.

The three roles:

```
┌──────────────────────────────────────────────────────────────────────┐
│                                 MVC                                    │
│                                                                        │
│   ┌────────────┐   reads/writes   ┌────────────┐   renders   ┌───────┐ │
│   │   MODEL    │◄────────────────►│ CONTROLLER │────────────►│ VIEW  │ │
│   │ data+rules │                  │   glue     │             │display│ │
│   └────────────┘                  └─────▲──────┘             └───────┘ │
│         ▲                              │                               │
│         │                       input  │                               │
│      persists to                 from user                             │
│      the database                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

- **Model — data + the rules about it.** In this course the Model is where **Knex** lives. It knows the tables, the columns, and the business rules that belong to the data (e.g. "a post with no title is not valid", "comments only for published posts"). It knows *nothing* about HTTP. Given `Post.findRecent(10)`, it returns ten rows — it does not know or care that they will be shown in HTML.
  - *Example:* `Post.findById(id)` runs `knex("posts").where({ id }).first()`.
  - *Why it matters:* every piece of code that needs posts calls the **same** function. Fix a bug there once, and every page is fixed.

- **View — presentation only.** The View is the Handlebars/EJS template. It receives already-prepared data (called **locals**) and turns it into HTML. It must not query anything, and it must not decide *business* questions.
  - *Example:* `{{#each posts}}<li>{{title}}</li>{{/each}}`.
  - *Why it matters:* a designer can rewrite the template without understanding SQL, and the same data can be rendered as HTML, JSON, or email later.

- **Controller — orchestration.** The controller is the **Express route handler**. It reads what the request wants (`req.params`, `req.query`, `req.body`), decides what to call on the Model, then chooses how to answer (`res.render`, `res.json`, `res.redirect`).
  - *Example:* a `GET /posts` handler calls `Post.findRecent(10)` and passes the result to `res.render("posts/index")`.
  - *Why it matters:* the controller is where the *flow* is readable in one screen. If it is readable, the app is understandable.

**Separation of concerns** is the general principle underneath: each module should have **one reason to change**. In MVC, "the table schema changed" changes the Model only; "the design changed" changes the View only; "the URL contract changed" changes the Controller only. When those blurs, a one-line change forces you to open five files at once — and that is exactly the coupling MVC exists to prevent.

### 2. How MVC Maps to This Course (Full-Stack SSR)

This course builds a **server-side-rendered** app: the browser asks for a URL, and the server replies with a *complete* HTML page. MVC is the internal shape of that server. One request travels through the layers like this:

```
 Browser                Express app (Node)                                    PostgreSQL
   │                          │                                                  │
   │  GET /posts              │                                                  │
   ├─────────────────────────►│                                                  │
   │                          │  1. ROUTE matches METHOD+PATH                    │
   │                          │     app.get("/posts", postsController.index)      │
   │                          │                          │                        │
   │                          │  2. CONTROLLER runs      ▼                        │
   │                          │     reads req, calls Model                       │
   │                          │                          │                        │
   │                          │  3. MODEL queries        │                        │
   │                          │     Post.findRecent(10)  ├─── SQL via Knex ─────►│
   │                          │                          │◄── rows ──────────────┤
   │                          │  4. CONTROLLER gets rows │                        │
   │                          │     res.render("posts/index", { posts })          │
   │                          │                          │                        │
   │                          │  5. VIEW renders template │                       │
   │                          │     Handlebars -> HTML    │                       │
   │  6. HTML response        │                          ▼                        │
   │◄─────────────────────────┤   <html>…{{posts}}…</html>                        │
   │                          │                                                  │
   │  browser paints the page │                                                  │
```

Read top to bottom: **Route → Controller → Model → (HTTP to DB back to Model) → Controller → View → HTML.** The controller appears twice because it is the hub: it calls *out* to the Model, then calls *out* to the View. The View only ever talks to the controller; the Model only ever talks to the controller (and the database). Nothing skips a layer.

**Why the layering is drawn this way.** Consider what would happen if the template called the database directly. Now every render requires a live DB, tests need a database, the query is duplicated across pages, and a schema change touches every `.handlebars` file. The bridge — the controller — exists so that *data fetching* and *data display* never have to know each other. That is the entire benefit: **you can change one without the other.**

### 3. Folder Structure — The Discipline Made Physical

MVC is invisible unless the folders enforce it. This is the canonical tree for the course project:

```
my-express-app/
├── server.js                # entry: build app, mount routes, app.listen()
├── package.json
├── db/
│   ├── knexfile.js          # connection config (client, connection, migrations)
│   └── index.js             # exports a single configured knex instance
├── routes/
│   └── index.js             # maps URLs -> controller functions
├── controllers/
│   └── postsController.js   # req/res orchestration (THIN)
├── models/
│   └── post.js              # Knex queries + data rules (NO req/res)
├── services/                # (optional) business logic that spans models
│   └── postService.js
├── middlewares/
│   └── requireAuth.js       # cross-cutting guards, logging
├── views/
│   ├── layouts/
│   │   └── main.handlebars  # shared shell (template note 05)
│   └── posts/
│       └── index.handlebars # presentation only
└── public/                  # static assets served by express.static
    ├── css/app.css
    └── img/logo.svg
```

| Folder | Owns | Never contains |
|---|---|---|
| `routes/` | URL → controller wiring | business logic, SQL |
| `controllers/` | reading `req`, calling models, choosing the response | SQL, HTML strings, template markup |
| `models/` | Knex queries + data rules | `req`, `res`, `res.render` |
| `services/` | multi-model business logic | HTTP concerns |
| `middlewares/` | cross-cutting steps (auth, logging, errors) | page-specific logic |
| `views/` | presentation | queries, business decisions |
| `db/` | connection setup + migrations | request handling |

**Naming conventions.** `postsController.js` exports `{ index, show, create, update, destroy }` — the resource actions. `post.js` (singular) exports data functions named as *questions*: `findRecent`, `findById`, `create`. Route files mirror the resource: `posts.js` in `routes/`. The consistency is not cosmetic: when you know that "the `posts` index page is broken", you look in exactly three predictable files.

**Why controllers stay thin.** A thin controller is a *table of contents* for a request. Anyone opening `postsController.index` should see: read input → call model → render. If that function is 120 lines, the reader can no longer tell what the request *does* — and the logic inside is trapped where it can only be reached by a web request.

### 4. The Model Layer in an Express App

The Model is plain JavaScript plus **Knex** (the query builder covered in [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md)). It is **pure in the sense that it does not know HTTP**: no `req`, no `res`, no template names. That is precisely what makes it unit-testable — you can call it from a script or from a test file with no server running.

```js
// models/post.js — data access only. No req, no res.
const db = require("../db"); // the configured knex instance

const Post = {
  // a "question" the rest of the app is allowed to ask
  async findRecent(limit = 10) {
    return db("posts")
      .select("id", "title", "slug", "created_at")
      .orderBy("created_at", "desc")
      .limit(limit);
  },

  async findById(id) {
    return db("posts").where({ id }).first();
  },

  async create({ title, body }) {
    // a business rule that belongs to the DATA, not to HTTP
    if (!title || !title.trim()) throw new Error("title is required");
    const [row] = await db("posts")
      .insert({ title: title.trim(), body })
      .returning("*");
    return row;
  },
};

module.exports = Post;
```

**The service layer (introduction).** A Model is naturally *per-table*. But some business rules need several models or several steps. When that happens, put the logic in a **service** — a function that coordinates models — rather than stuffing it into the controller:

```js
// services/postService.js — business logic that spans models
const Post = require("../models/post");
const Comment = require("../models/comment");

async function getPostWithCommentCount(id) {
  const post = await Post.findById(id);
  if (!post) return null; // "not found" is a business answer, expressed as data
  const [{ count }] = await Comment.countForPost(id);
  return { ...post, commentCount: Number(count) };
}

module.exports = { getPostWithCommentCount };
```

The rule of thumb: **one table → a Model; a workflow across tables → a Service; request plumbing → a Controller.** Keeping the service free of `req`/`res` is what lets you reuse it from a CLI job, a cron, or an API later.

### 5. The View Layer

The View is the template engine file (**Handlebars** or **EJS**; the dedicated note is [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md)). It receives **locals** — the object passed as the second argument to `res.render` — and turns them into HTML.

```handlebars
{{!-- views/posts/index.handlebars — presentation only --}}
<h1>Recent Posts</h1>

{{#if posts.length}}
  <ul class="post-list">
    {{#each posts}}
      <li>
        {{!-- linking routes: the View only builds URLs, it does not fetch --}}
        <a href="/posts/{{this.id}}">{{this.title}}</a>
        <small>{{this.created_at}}</small>
      </li>
    {{/each}}
  </ul>
{{else}}
  <p>No posts yet.</p>
{{/if}}

<a href="/posts/new">Write a post</a>
```

Three rules make this a View and not a second controller:

1. **Only presentation logic.** `{{#if}}`/`{{#each}}` decide *how* to draw — never *whether a user may see a post* (that is authorization, a controller/middleware job).
2. **No data access.** A template must never call `db(...)`. If it needs data, the controller must have supplied it.
3. **Links, not fetches.** The View emits `<a href="/posts/{{id}}">`; the browser then makes a *new* request that re-enters the routing layer. This is the SSR navigation model from [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md).

### 6. Implementation (Code) — A Minimal, Runnable MVC Seed

This is the smallest complete MVC slice. Run it and `GET /posts` produces a rendered page from a real Knex query.

```js
// db/knexfile.js — connection config (kept out of the app logic)
module.exports = {
  development: {
    client: "pg",
    connection: process.env.DATABASE_URL || {
      host: "127.0.0.1", database: "diary_dev", user: "postgres", password: "postgres",
    },
    migrations: { directory: "./migrations" },
  },
};
```

```js
// db/index.js — ONE configured connection, imported everywhere
const knex = require("knex");
const config = require("./knexfile")[process.env.NODE_ENV || "development"];
module.exports = knex(config);
```

```js
// models/post.js
const db = require("../db");
const Post = {
  findRecent: (limit = 10) =>
    db("posts").select("id", "title", "created_at").orderBy("created_at", "desc").limit(limit),
};
module.exports = Post;
```

```js
// controllers/postsController.js — THIN: read, call model, respond
const Post = require("../models/post");

exports.index = async (req, res, next) => {
  try {
    const posts = await Post.findRecent(10);      // 1. call the Model
    res.render("posts/index", { posts, title: "Recent Posts" }); // 2. hand to the View
  } catch (err) {
    next(err);                                     // 3. never swallow errors
  }
};
```

```js
// routes/index.js — URL -> controller wiring, nothing else
const express = require("express");
const router = express.Router();
const postsController = require("../controllers/postsController");

router.get("/posts", postsController.index);

module.exports = router;
```

```js
// server.js — the entry point that assembles everything
const express = require("express");
const { engine } = require("express-handlebars");
const routes = require("./routes");

const app = express();

app.engine("handlebars", engine());
app.set("view engine", "handlebars");
app.set("views", "./views");

app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));
app.use("/", routes);

// centralized error handler (one place, four args) — see note 04
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).render("errors/500", { message: err.message });
});

app.listen(3000, () => console.log("MVC app on http://localhost:3000"));
```

**Reading the chain:** `server.js` configures the engine and mounts `routes/index.js`; the route points `/posts` at `postsController.index`; the controller asks `Post.findRecent`; the model runs a Knex query; the rows come back and go into `res.render("posts/index", { posts })`; the View loops and emits HTML. Change the table name and you edit **only** `models/post.js`. Change the markup and you edit **only** the `.handlebars` file. That is MVC working.

### 7. Variants — Fat Controllers, MVP, and MVVM

**The fat-controller anti-pattern.** MVC degrades the moment the controller starts doing everyone's job. The classic descent:

```js
// ❌ FAT CONTROLLER — everything in one place
app.get("/posts", async (req, res) => {
  const rows = await db("posts").orderBy("created_at", "desc").limit(10); // SQL in controller
  if (req.session.user?.role !== "admin") { /* auth scattered inline */ }
  const html = "<ul>" + rows.map(p => `<li>${p.title}</li>`).join("") + "</ul>"; // HTML in controller
  res.send(html);
});
```

It *works* — and that is what makes it dangerous. It is impossible to test without HTTP, impossible to restyle without editing logic, and every later feature (pagination, authorization, a JSON API) has to wedge itself into the same function. The fix is not a framework; it is moving each line back to its layer.

**MVC vs MVP vs MVVM.** All three split a system, but they differ in *who talks to whom*:

- **MVP (Model–View–Presenter):** the **Presenter** holds all view logic and talks to the View through an interface; the View is passive. Common in classic desktop/Android UI.
- **MVVM (Model–View–ViewModel):** the **ViewModel** exposes data that the View *binds* to automatically; changes flow both ways without explicit "set text" calls. This is the model behind Vue, Angular, and much of React's ecosystem.
- **MVC in a server-SSR context:** the View is a one-way template (no live binding), and the controller is request-scoped rather than persistent. This is exactly why MVC fits SSR and MVVM fits rich client apps.

| Aspect | MVC (this course) | MVP | MVVM |
|---|---|---|---|
| Middle layer | Controller | Presenter | ViewModel |
| View↔middle link | One-way template render | Presenter calls View via interface | Two-way data binding |
| Best fit | Server-side rendering | Desktop/Android UI | Reactive client apps (Vue/Angular) |
| Testable without UI | Yes (controllers + models) | Yes (presenter) | Yes (viewmodel) |

**Thin controller vs fat controller — consequences:**

| Concern | Thin controller | Fat controller |
|---|---|---|
| Testability | Unit-test the model directly; controller is trivial | Needs a running server and DB to test anything |
| Reuse | Model reusable by API, cron, tests | Logic locked inside one route |
| Change | Restyle touches views only | A UI change forces edits to logic |
| Merge conflicts | Different concerns, different files | Everyone edits the same mega-function |
| Readability | Function reads like a list of steps | Reader must reverse-engineer intent |

---

## ⚠️ Common Pitfalls

| Pitfall | Symptom | Why | Fix |
|---|---|---|---|
| HTML built inside the controller | `res.send("<h1>…")` strings everywhere | Presentation leaked into orchestration | Move markup to a template; `res.render` |
| SQL inside the controller | `db("posts")…` in a route handler | No Model boundary; query duplicated | Move queries into `models/`; controller calls the model |
| View reaches into the DB | Template awaits a query or receives `db` | View now needs a database to render | Controller supplies all data as locals |
| God controller | One file with 40 handlers, 800 lines | No split by resource | One controller per resource (`postsController`, `usersController`) |
| Forgetting `next(err)` in async | Unhandled rejection, request hangs | Express 4 does not catch async throws automatically | `try/catch` + `next(err)`, or a wrapper — see [`./04-routing-in-express.md`](./04-routing-in-express.md) |
| Business rule in the wrong layer | Validation duplicated in controller *and* template | Rule has no single owner | Put it in the Model/Service; both layers consume the same answer |
| Model imports `res`/`req` | Model signature takes `(req, res)` | HTTP leaked into data layer; untestable | Model takes plain arguments, returns plain data |

**Why it matters.** This is the **shape of the course project** — every later note (templates, sessions, auth, uploads) plugs into these seams. But the discipline outlives the course: React/Next components are "views", hooks and stores are "controllers/viewmodels", and data-fetching modules are "models". An AI service layer follows the same rule — the model inference code should not know about the HTTP request that triggered it. MVC is the first place you practice the habit that keeps large systems changeable.

---

## 🔗 Tech Cross-References
- **[JavaScript language core]** — the module exports (`module.exports`), object-literal "objects", and `async/await` used to write models and controllers. See [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md).
- **[Databases / Knex]** — the Model layer is a Knex query builder + connection; this note assumes that machinery. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **[Web Security / authorization]** — an auth check is a *middleware guard* that runs before the controller, not a line inside every handler. See [`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md).
- **[Software architecture]** — MVC's Model/View/Controller is a *layering* pattern; the tiers (presentation / application / data) generalise it at system scale. See [`../../software_architecture/03-application-architecture-models.md`](../../software_architecture/03-application-architecture-models.md).

## 🧪 Practice Tasks
> No solutions provided. Do them, then ask the AI to grade the evidence (code + `curl -i` output).

- [ ] **Task 1 — Purify a model.** Take any query currently written inline in a route handler and move it into a `models/` function with plain arguments and a plain return value. Prove it by calling that function from a standalone `node -e` script with no server running.
- [ ] **Task 2 — Extract a service.** Write a `postService.getPostWithCommentCount(id)` that coordinates two models. Then write a controller that uses it. Success = the controller contains zero SQL and zero HTML.
- [ ] **Task 3 — Refactor a fat controller.** Given a 60-line inline handler that queries, validates, authorizes, and builds HTML, refactor it into route + controller + model + service + view. Draw the before/after dependency arrows.
- [ ] **Task 4 (harder) — Swap the renderer.** Make the *same* controller data available as HTML (`res.render`) and as JSON (`res.json`) selected by `req.query.format`. Explain, in one sentence, why MVC made this cheap.

## 📝 Research Journey
- **Why:** I could already write Express routes that *work*, but every tutorial's "small demo" and every "real project" looked nothing alike — the demo was three lines, the project was folders of models and controllers and services. I wanted to know what actually forces that structure, not just copy it.
- **Struggle:** My first instinct was that MVC was ceremony: why call `Post.findRecent()` when I can just write `db("posts")` right there in the route? It felt like extra files for no gain. The word "concern" meant nothing to me until I hit the real pain — a constant "I just need this one small change" that somehow required opening five files, or a test I literally could not write without spinning up the server.
- **Aha moment:** When I finally wrote a script that imported `models/post.js` and called `findRecent()` **with no web server at all**, and it just ran. The Model did not care that it was being called from a test, a CLI, or a route. That is when "separation of concerns" stopped being a phrase and became a feeling: each layer could be exercised alone.
- **Career link:** An AI Engineer's work is 20% modelling and 80% plumbing the model into a system. When I later wire an inference function into a server, the exact same question appears: does the model code know about HTTP? The MVC discipline I build here is the discipline that keeps an ML service testable — the prediction function should be callable from a notebook, a test, and a route, unchanged.

## 🔗 Related Topics
- [`./04-routing-in-express.md`](./04-routing-in-express.md) — the routing + middleware machinery the Controller sits inside.
- [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) — the View layer in full (layouts, partials, helpers).
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the query builder powering every Model.
- [`../../software_architecture/03-application-architecture-models.md`](../../software_architecture/03-application-architecture-models.md) — how MVC generalises into system-wide tiering.
- [SUGGESTED] `09-services-and-dependency-injection.md` — when a Service layer needs to receive its Models (injection) instead of importing them, to become testable with fakes. *Important because:* the moment you want to test a service without a database, you meet dependency injection — the natural next step after this note.
- [SUGGESTED] `bridge-mvc-to-nextjs-components.md` — maps Controller/View/Model onto Next.js server components, route handlers, and data modules. *Important because:* it connects this SSR course to the Joblink (Next.js) stack you will read next.

## 🤔 Open Questions
- [ ] Where exactly does **validation** live — Model, Service, or Controller — when the same rule must guard both a web form and a future JSON API?
- [ ] At what project size does a Service layer earn its keep, and when is it premature abstraction over a handful of models?
- [ ] How do transactions (multi-step writes) fit the Model/Service boundary — does the Service own the transaction, or the Model?
- [ ] Is MVC still the right name in an SSR app where the View is a one-way template and there is no persistent controller per UI element?

---
*Separate what changes for different reasons. The route reads, the model knows, the view draws — and none of them does the others' job.*

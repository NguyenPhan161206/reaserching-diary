# Advanced Features — Pagination, WYSIWYG & File Upload

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Pagination`, `#AJAX`, `#FileUpload`, `#multipart`, `#multer`, `#WYSIWYG`, `#Sanitize`, `#Express`, `#Knex`
**References:** [Knex — `limit`/`offset`/`count`](https://knexjs.org/guide/query-builder.html) · [multer (npm)](https://www.npmjs.com/package/multer) · [Express — `express.static`](https://expressjs.com/en/starter/static-files.html) · [sanitize-html (npm)](https://www.npmjs.com/package/sanitize-html) · [TinyMCE — Quick start](https://www.tiny.cloud/docs/tinymce/latest/quick-start/) · [Quill](https://quilljs.com/docs/quickstart/) · [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
**Status:** Draft
**Difficulty:** Intermediate → Advanced
**Prerequisites:** [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md) (forms, `FormData`, multipart), [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) (CRUD), [`./04-routing-in-express.md`](./04-routing-in-express.md) (routes/middleware), [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) (views)

---

## 📌 Key Takeaways (TL;DR)
- These are **three "feature adds" bolted onto the CRUD SSR app**, and each is a *client + server pair*: "one page pattern + one POST/upload flow + one security rule." Learn them as a shape, not three unrelated topics.
- **Pagination is a SQL concern** (`LIMIT`/`OFFSET` + a `COUNT(*)` for the total), not a "show 10 things" loop in JavaScript. The page links are generated from the total and the page size.
- **Ajax pagination is progressive enhancement**: the `<a href="/posts?page=2">` links must work without JavaScript; JS only intercepts them to swap the list node instead of reloading.
- **The client form must set `enctype="multipart/form-data"`** or a file never rides the wire. On the server, **`multer`** parses that multipart body.
- **Never trust a user filename for a path.** Generate the stored name yourself (`crypto.randomUUID()`) and whitelist the type — that single rule kills path traversal and executable uploads.
- **A WYSIWYG editor posts raw HTML — storing and re-rendering it is stored XSS.** Sanitize server-side (`sanitize-html`) *before* persisting, and only then render raw (`{{{ }}}`).
- **Wire full-text search in the view here; the SQL lives in [`../databases/07-postgresql-full-text-search.md`](../databases/07-postgresql-full-text-search.md).** This note only connects the box to the query.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

A working CRUD app is a list page, a create form, and a detail page. The course's "advanced features" chapter bolts three upgrades onto that skeleton — and each one is the same *shape*:

```
   FEATURE            CLIENT (browser)                SERVER (Express)
   ────────           ────────────────                ────────────────
   Pagination    ──►   page links / ?page=2      ──►   LIMIT/OFFSET + COUNT(*)
   Ajax page     ──►   fetch() swaps list node    ──►   JSON endpoint
   Upload        ──►   <form enctype=multipart>   ──►   multer → disk → static serve
   WYSIWYG       ──►   editor posts HTML          ──►   sanitize → store → render
```

**Why "feature add" is the right mental model:** none of these changes the architecture — they are still `route → controller → model → view` (see [`./04-routing-in-express.md`](./04-routing-in-express.md)). What changes is one decision per feature: *which query?*, *which response shape?*, *which file rules?*, *which sanitizer?*. Once you see them as plug-ins on the same MVC spine, the "advanced" label stops being intimidating.

**The shared security thread:** every one of the three touches a boundary where untrusted input crosses into your system — a query parameter, a JSON body, an uploaded file, an HTML string. So each one ends with a security rule (parameterised query, escape output, whitelist file type, sanitize HTML). The client half lives in [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md); this note is the server half.

---

### 2. Server-Side Pagination

Listing every row is fine for 20 posts and fatal for 200,000 — you read the whole table into memory and render an unusable page. Pagination means **read one page's worth at a time and tell the user how many pages exist**.

Two numbers drive everything:

- `page` — which slice, 1-based, from the query string (`?page=3`).
- `perPage` — the page size, a constant you choose (say 10).

The math: total pages $= \lceil \text{total} / \text{perPage} \rceil$, and the SQL offset $= (\text{page} - 1) \times \text{perPage}$.

```js
// models/posts.js
const db = require("../db");
const PER_PAGE = 10;

async function listPosts(page = 1) {
  const requested = Math.max(1, parseInt(page, 10) || 1);      // guard NaN/0/negative
  const [{ count }] = await db("posts").count("id as count");  // total rows for page math
  const total = Number(count);
  const totalPages = Math.max(1, Math.ceil(total / PER_PAGE));
  const current = Math.min(requested, totalPages);             // clamp out-of-range

  const rows = await db("posts")
    .select("id", "title", "created_at")
    .orderBy("created_at", "desc")                             // ORDER is mandatory for a stable slice
    .limit(PER_PAGE)
    .offset((current - 1) * PER_PAGE);

  return { rows, total, totalPages, page: current, perPage: PER_PAGE };
}
```

**Why `ORDER BY` is mandatory:** `LIMIT`/`OFFSET` slice *some* arbitrary part of the result unless you sort. Without a deterministic order, page 2 can repeat items from page 1 — a classic "why do I see the same row twice?" bug.

**Why `COUNT(*)` separately:** `LIMIT` tells you how many rows you got, not how many exist. You need the total to render "Page 3 of 17" and to know whether Next should appear. Two queries, one job each.

**Edge cases this handles:**
- `page=0` or `page=-5` → clamped to 1 (`Math.max`).
- `page=abc` → `parseInt` yields `NaN` → fallback 1.
- `page=9999` past the end → clamped to `totalPages`, so you never render an empty page with a broken pager.

**The performance cliff:** `OFFSET 1000000` makes Postgres *scan and discard* a million rows before returning yours — deep pages get linearly slower. That is a database-tuning problem (keyset/cursor pagination, covering indexes) covered in [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md); for a course project, offset pagination is correct and cheap.

**Rendering the pager in the view** (Handlebars-style; build the page-number array in the controller):

```hbs
<nav class="pager">
  {{#if (gt page 1)}}<a href="/posts?page={{dec page}}">Prev</a>{{/if}}
  {{#each pages}}
    <a href="/posts?page={{this}}" class="{{#if (eq this ../page)}}active{{/if}}">{{this}}</a>
  {{/each}}
  {{#if (lt page totalPages)}}<a href="/posts?page={{inc page}}">Next</a>{{/if}}
</nav>
```

*Why it matters:* the pager is **derived state** — it comes from `total`/`page`, never stored. Get the two numbers right and the links fall out; get them wrong and every page is off by one.

---

### 3. Ajax Pagination (Progressive Enhancement)

Classic pagination reloads the whole page per click. Ajax pagination sends the *same* page request to a JSON endpoint and swaps only the list node — the header, sidebar, and scroll position stay put.

```
   WITHOUT JS  (baseline)          WITH JS  (enhancement)
   ─────────────────────           ──────────────────────
   click <a href=...page=2>        click → preventDefault
        │                               │
        ▼                               ▼
   full page reload              fetch('/api/posts?page=2')
   server renders new HTML             │
                                       ▼
                                  res.json({rows, page, ...})
                                       │
                                       ▼
                                  list.innerHTML = render(rows)
```

**Server: a JSON twin of the list route.** Reuse the exact same model function — one source of truth for the page math:

```js
// routes/api.js
router.get("/api/posts", async (req, res, next) => {
  try {
    const data = await listPosts(req.query.page);   // identical to the SSR route
    res.json(data);
  } catch (err) { next(err); }
});
```

**Client: intercept the link, swap the container, update history.** The links still have real `href`s, so the page works with JS disabled:

```html
<div id="list">{{#each rows}}<article>…</article>{{/each}}</div>
<nav id="pager">…real <a href="/posts?page=2"> links…</a></nav>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const pager = document.getElementById("pager");
  pager.addEventListener("click", async (e) => {
    const link = e.target.closest("a[href^='/posts?page=']");
    if (!link) return;
    e.preventDefault();                              // take over from the browser
    const page = new URL(link.href).searchParams.get("page");

    const res = await fetch(`/api/posts?page=${page}`);
    if (!res.ok) return;                             // fetch does NOT throw on 404/500
    const data = await res.json();

    document.getElementById("list").innerHTML =
      data.rows.map((r) => `<article>${escapeHtml(r.title)}</article>`).join("");
    document.getElementById("pager").innerHTML = buildPager(data);  // re-render state
    history.pushState({ page }, "", `/posts?page=${page}`);         // URL stays shareable
  });
});
</script>
```

**Why progressive enhancement is the correct architecture:** the no-JS path is the *contract* (a real page request), and JS is an optimization layered on top. This means the app is accessible, crawlable, and debuggable with `curl` — and if your `fetch` code breaks, users still browse. The opposite (JS-only) makes a page that is blank without a script, which is a failure mode, not a design.

**The two Ajax gotchas:** (1) re-render the *pager* too, or Prev/Next/active-state go stale; (2) escape any data you inject via `innerHTML` (a tiny `escapeHtml`) — otherwise this becomes the stored-XSS vector from §5 at the client edge.

---

### 4. File Upload (Multipart)

A file cannot travel as normal form text; it needs `multipart/form-data` (the browser adds a boundary — see [`../web_basics/08`](../web_basics/08-forms-fetch-and-multipart-upload.md)). On the server, **`multer`** parses that multipart body into `req.file` / `req.files`.

**The client form — the `enctype` is not optional:**

```html
<form method="post" action="/posts" enctype="multipart/form-data">
  <input name="title" required>
  <input type="file" name="cover" accept="image/*">
  <button type="submit">Create</button>
</form>
```

Omit `enctype` and the file arrives as `undefined` (or a string), never as bytes. That is the single most common upload bug.

**The server — storage, filter, limits, handler:**

```js
const multer = require("multer");
const crypto = require("crypto");
const path = require("path");

const EXT_BY_MIME = { "image/jpeg": ".jpg", "image/png": ".png", "image/webp": ".webp" };

const storage = multer.diskStorage({
  destination: "public/uploads",                       // where bytes land
  filename: (req, file, cb) => {
    // IGNORE file.originalname for the path — derive the extension from the MIME type
    const ext = EXT_BY_MIME[file.mimetype] || "";
    cb(null, `${crypto.randomUUID()}${ext}`);          // random, collision-proof, safe
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 2 * 1024 * 1024, files: 1 },      // 2 MB, one file
  fileFilter: (req, file, cb) => {
    if (!EXT_BY_MIME[file.mimetype]) return cb(new Error("Unsupported file type"));
    cb(null, true);                                     // whitelist by MIME, reject the rest
  },
});

router.post("/posts", requireAuth, upload.single("cover"), async (req, res, next) => {
  if (!req.file) return res.status(400).send("No file uploaded");
  const url = `/uploads/${req.file.filename}`;
  await db("posts").insert({
    title: req.body.title,
    cover_url: url,
    author_id: req.session.userId,
  });
  res.redirect("/posts");
});
```

**Serving the files — `express.static`:**

```js
app.use("/uploads", express.static("public/uploads"));  // GET /uploads/<name>
```

**The security rules, and why each exists:**

| Rule | Attack it stops |
|---|---|
| **Generate random filenames** (`crypto.randomUUID()`) | Path traversal — a filename like `../../app.js` escaping the upload dir; also overwriting existing files |
| **Whitelist extension/MIME** in `fileFilter` | Uploading `.php`, `.html`, `.svg`, `.sh` — an executable/script that runs or renders |
| **Size limit** (`limits.fileSize`) | Disk exhaustion / DoS from a giant upload |
| **Store outside the web root, serve via a mapped path** | Direct execution — uploads should be data, not code |
| **Serve uploads from a separate domain/sub-path** | Stored XSS: a `.html`/`.svg` upload served same-origin executes as *your* site |

**Why `file.originalname` is never a path:** the client controls it completely — it can contain `../`, an absolute path, null bytes, or a `.php` extension. Trusting it is how upload folders become remote-code-execution footholds. Two defenses together: **random name** + **MIME whitelist** (never guess the type from the extension the user supplied).

**Why memory vs disk matters:** `multer.diskStorage` streams to disk (good for large files, survives between requests); `multer.memoryStorage` holds the buffer in RAM (good for small files you immediately push to S3 or process) — but memory storage plus a large `fileSize` limit is a DoS waiting to happen.

---

### 5. WYSIWYG Editor (Rich Text → Stored XSS Risk)

A WYSIWYG ("what you see is what you get") editor replaces a plain `<textarea>` with a toolbar: bold, italic, links, lists. Mechanically it is simple — **embed the editor over a textarea, and on submit it writes HTML into that textarea's value** — and that is exactly the danger.

**Client — TinyMCE (one script + one init):**

```html
<textarea name="body" id="body"></textarea>
<script src="https://cdn.tiny.cloud/1/no-api-key/tinymce/6/tinymce.min.js" referrerpolicy="origin"></script>
<script>
  tinymce.init({
    selector: "#body",
    height: 400,
    plugins: "lists link code",
    toolbar: "undo redo | bold italic | bullist numlist | link | code",
  });
</script>
```

Quill works the same way (a container div + a hidden input). Either way: **the POST body contains raw HTML**, e.g. `<p>Hello <b>world</b></p>` — but also possibly `<script>…</script>` or `<img src=x onerror=…>`.

**The danger — stored XSS.** If you save that HTML verbatim and later render it unescaped, every visitor executes the attacker's script in your origin's context: it can steal the session cookie (if not `HttpOnly`), act as the user, or rewrite the page. This is *stored* (persistent) XSS, the worst kind, because it fires for every reader. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)

**The defense — sanitize on the server, before persisting:**

```js
const sanitizeHtml = require("sanitize-html");

const cleanBody = sanitizeHtml(req.body.body, {
  allowedTags: ["p", "br", "strong", "em", "ul", "ol", "li", "a", "h2", "h3", "blockquote"],
  allowedAttributes: { a: ["href", "title"] },
  allowedSchemes: ["http", "https", "mailto"],        // blocks javascript: URLs
});

await db("posts").insert({ title: req.body.title, body: cleanBody });
```

**The rendering decision in the template — escaped vs raw.** Handlebars escapes by default:

- `{{ body }}` → HTML-escaped (`<b>` shows as text). Safe, but the rich text renders as tags — useless.
- `{{{ body }}}` → raw HTML injected. Needed for formatting, but **only safe because it was sanitized on write**.

The rule: **sanitize on the way *in*, then render raw on the way *out*.** Sanitizing at render time also works but is slower and easier to forget on one code path; sanitizing at write time guarantees the stored value is already safe. Never render raw HTML you have not sanitized — that single sentence is the whole lesson.

**Why server-side, not client-side:** an attacker posts directly with `curl`, bypassing your editor and any JS sanitizer entirely. Like all validation, sanitization is only trustworthy on the server.

---

### 6. Combined Mini-Project Sketch

Putting all three together — an admin posts manager:

```
   GET /admin/posts            ──► listPosts(page) ──► list.hbs
        │  (pagination + ajax pager)
        │
   click "New post"            ──► new.hbs  (TinyMCE + <input type=file>)
        │
   POST /admin/posts           ──► requireAuth (07)
        │                          upload.single("cover")   ← multer: filter+limit+random name
        │                          sanitizeHtml(req.body.body)
        │                          db("posts").insert({ title, body, cover_url, author_id })
        ▼
   redirect /admin/posts       ──► list shows new row with cover image
```

Walk one request end to end:

1. **Route** `POST /admin/posts` matched, `requireAuth` passes (session from note 07).
2. **Multer middleware** parses the multipart body; `fileFilter` rejects a `.php`, `limits` rejects >2 MB, storage writes `public/uploads/<uuid>.png`.
3. **Controller** reads `req.body.title`, `req.body.body` (editor HTML), `req.file.filename`.
4. **Sanitize** `req.body.body` with `sanitize-html` — the stored value can no longer contain scripts.
5. **Model** inserts one row via Knex, storing `/uploads/<uuid>.png` as `cover_url`.
6. **Redirect** to the list; the next `GET` paginates and renders. The detail view uses `{{{ body }}}` (safe, because step 4 ran) and `<img src="{{cover_url}}">` (escaped, as URLs always should be).

**The shape to remember:** every feature is *route → guard → parser → sanitize/validate → model → view*. The "advanced" parts are just extra boxes in the middle of that pipeline.

---

## 🧾 Command / Code Cheat Sheet

| Command / Code | What it does | When to use | Example / Output |
|---|---|---|---|
| `db("posts").count("id as count")` | Total row count for page math | Before rendering any pager | `[{ count: "137" }]` |
| `.limit(10).offset((page-1)*10)` | Read one page's slice | Every paginated list query | rows 11–20 for page 2 |
| `.orderBy("created_at", "desc")` | Deterministic slice order | **Always** with limit/offset | stable pages, no repeats |
| `res.json({rows, page, totalPages})` | JSON twin of a list route | Ajax pagination endpoint | `{"page":2,"rows":[…]}` |
| `multer.diskStorage({destination, filename})` | Write uploads to disk with a chosen name | Any file upload | `public/uploads/<uuid>.png` |
| `fileFilter: (req,file,cb) => …` | Whitelist MIME types | **Always** — reject the rest | `Unsupported file type` |
| `limits: { fileSize: 2*1024*1024 }` | Cap upload size | Public uploads | `413` / multer error |
| `upload.single("cover")` | Parse one file field named `cover` | Single-image form | `req.file.filename` |
| `express.static("public/uploads")` | Serve uploaded files | Expose `/uploads/*` | `GET /uploads/x.png` |
| `crypto.randomUUID()` | Collision-proof safe filename | Every stored upload | `f47ac10b-…` |
| `tinymce.init({ selector: "#body", … })` | Turn a textarea into a rich editor | Rich-text create form | posts HTML on submit |
| `sanitizeHtml(html, { allowedTags })` | Strip dangerous tags before storing | **Always** on editor HTML | `<script>` removed |
| `{{{ body }}}` vs `{{ body }}` | Raw vs escaped render | Raw only for sanitized HTML | formatting vs literal tags |

> **Rule:** if a line touches untrusted input (a page number, a file, an HTML string), the "when to use" is *always* — and the adjacent sanitize/validate/limit line is mandatory, not optional.

---

## 🧪 Practical Lab

> A runnable mini-app: a paginated list plus an upload route storing to `/uploads`, printing the stored filename. Verify against the expected output.

```bash
mkdir /tmp/adv-lab && cd /tmp/adv-lab
npm init -y
npm i express express-handlebars knex pg multer
```

```js
// app.js
const express = require("express");
const multer = require("multer");
const crypto = require("crypto");
const path = require("path");

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use("/uploads", express.static("public/uploads"));

// --- fake data layer (swap for Knex in the real project) ---
const POSTS = Array.from({ length: 23 }, (_, i) => ({
  id: i + 1, title: `Post ${i + 1}`, created_at: new Date(),
}));
const PER_PAGE = 5;
function listPosts(page = 1) {
  const requested = Math.max(1, parseInt(page, 10) || 1);
  const total = POSTS.length;
  const totalPages = Math.max(1, Math.ceil(total / PER_PAGE));
  const current = Math.min(requested, totalPages);
  const rows = POSTS.slice((current - 1) * PER_PAGE, current * PER_PAGE);
  return { rows, total, page: current, totalPages, perPage: PER_PAGE };
}

// --- server-side pagination (HTML) ---
app.get("/posts", (req, res) => {
  const data = listPosts(req.query.page);
  res.type("html").send(
    `<ul>${data.rows.map((p) => `<li>${p.title}</li>`).join("")}</ul>` +
    `<p>Page ${data.page} of ${data.totalPages}</p>`
  );
});

// --- ajax pagination (JSON) ---
app.get("/api/posts", (req, res) => res.json(listPosts(req.query.page)));

// --- upload ---
const storage = multer.diskStorage({
  destination: "public/uploads",
  filename: (req, file, cb) => cb(null, `${crypto.randomUUID()}.png`),
});
const upload = multer({ storage, limits: { fileSize: 2 * 1024 * 1024 } });
app.post("/upload", upload.single("cover"), (req, res) => {
  if (!req.file) return res.status(400).json({ error: "no file" });
  console.log("stored:", req.file.path);
  res.json({ ok: true, url: `/uploads/${req.file.filename}` });
});

app.listen(3000, () => console.log("on http://localhost:3000"));
```

```bash
mkdir -p public/uploads && node app.js
```

**Expected output (verify each):**

```bash
curl -s "http://localhost:3000/posts?page=2"
# <ul><li>Post 6</li>…<li>Post 10</li></ul><p>Page 2 of 5</p>

curl -s "http://localhost:3000/posts?page=99"        # clamped, not empty
# <ul><li>Post 21</li>…<li>Post 23</li></ul><p>Page 5 of 5</p>

curl -s "http://localhost:3000/api/posts?page=1"
# {"rows":[{"id":1,"title":"Post 1",…}],"total":23,"page":1,"totalPages":5,…}

curl -s -F "cover=@/path/to/pic.png" http://localhost:3000/upload
# {"ok":true,"url":"/uploads/<uuid>.png"}
# server terminal:  stored: public/uploads/<uuid>.png

ls public/uploads
# <uuid>.png        ← the random name, never the original filename
```

*What to notice:* `page=99` returns the **last** page, not an empty one — that is the clamp. The stored filename is a **UUID**, proving you never trusted the uploaded name. And `/api/posts` returns the *same* `{rows, totalPages, page}` shape the HTML route used, so the client can re-render both list and pager.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---|---|---|---|
| Deep `OFFSET` slowness | Page 10,000 takes seconds | Postgres scans and discards all skipped rows | Use keyset/cursor pagination for deep pages [→ `databases/08`](../databases/08-db-bottlenecks-indexing-and-caching.md) |
| Forgetting `COUNT(*)` | No "page X of Y", Next always shown | `LIMIT` gives rows, not the total | Add a separate `count()` query |
| No `ORDER BY` with `LIMIT` | Same row on two pages, others missing | Unordered slice is arbitrary | Always `orderBy` a stable column |
| Multer without `fileFilter` | A `.php`/`.html`/`.svg` lands in uploads | No type check; the browser/server may execute it | Whitelist MIME types; reject everything else |
| Trusting `file.originalname` | Path traversal, overwrites | The client controls the name completely | Generate `crypto.randomUUID()`; ignore the original |
| Raw HTML WYSIWYG stored | Stored XSS on every page view | Editor posts `<script>`/`onerror` payloads | `sanitize-html` before persist; raw render only after |
| Ajx `fetch` before DOM ready | `Cannot read properties of null` | Script ran before elements existed | Wrap in `DOMContentLoaded`, or use `defer` |
| `FormData`/file with no `enctype` | File arrives `undefined` | Browser sent urlencoded, not multipart | `enctype="multipart/form-data"` |
| Hand-setting `Content-Type` on multipart | `Boundary not found` on server | You erased the auto boundary | Omit the header; let the browser set it |
| Serving uploads same-origin unvalidated | Uploaded `.html`/`.svg` runs as your site | Same-origin execution | Whitelist types + serve from a separate subpath/domain |
| `innerHTML` with server data in Ajax | XSS via the JSON list | Unescaped injection | Escape before injecting, or use `textContent` |

---

## 🔗 Tech Cross-References

- **Databases (Knex & PostgreSQL)** — pagination *is* `LIMIT`/`OFFSET`/`COUNT`; the model layer is the same Knex CRUD from [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **Full-text search (wire only)** — the search box on this list page hits the SQL from [`../databases/07-postgresql-full-text-search.md`](../databases/07-postgresql-full-text-search.md); this note only connects `req.query.q` to that query and renders results.
- **Client upload mechanics** — the `FormData`/multipart/`fetch` half is fully explained in [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md).
- **Web security** — upload XSS, stored XSS from the editor, and escaping belong to [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md); the guards used here come from [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md).
- **Reverse proxy / Nginx** — in production, static uploads are usually served by Nginx directly, and `client_max_body_size` caps upload size at the edge. See [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md).

## 🧪 Practice Tasks
> No solutions. Do them, then ask the AI to grade your evidence (the SQL, the Network tab, the stored files).

- [ ] **Task 1 — Paginate for real.** With the course DB, add `?page=` to a list route using Knex `count()` + `limit()` + `offset()`. Prove the clamp works by requesting `page=0`, `page=abc`, and a page past the end, and show the pager links generated for each.
- [ ] **Task 2 — Add Ajax on top.** Give the list page a JSON twin endpoint and a `fetch` handler that swaps the list on click. Capture the Network request and demonstrate that disabling JavaScript still navigates correctly (progressive enhancement).
- [ ] **Task 3 — Harden an upload.** Build `POST /upload` with multer. Then try to break it: upload a `.php`, an oversized file, and a file named `../../evil.js`. Record the exact rejection for each and the stored filename after a valid upload.
- [ ] **Task 4 (harder) — Sanitize the editor.** Embed TinyMCE, post a body containing `<script>alert(1)</script>` and `<img src=x onerror=alert(2)>`. Show the raw stored value *before* sanitation, then the cleaned value after `sanitize-html`, and explain why `{{{ body }}}` is only safe post-sanitize.

## 📝 Research Journey
- **Why:** These are the last three features of the course project, and they are the ones that make it feel like a *product* rather than a CRUD exercise — real sites paginate, real sites let you write formatted posts, real sites let you upload an image. I wanted to understand them as one connected system, not three copy-paste snippets.
- **Struggle:** The WYSIWYG section flipped a switch for me. I had mentally filed XSS under "someone else's bug", but a rich-text editor *literally asks the user to submit HTML* — it is an XSS delivery mechanism by design unless you sanitize. Seeing that `{{ body }}` and `{{{ body }}}` differ by one brace, and that the difference is the entire safety of the feature, was uncomfortable in a useful way.
- **Aha moment:** Realizing pagination is a **SQL** problem, not a front-end one. Once I wrote `COUNT(*)` alongside `LIMIT`/`OFFSET`, the pager stopped being mysterious state and became arithmetic derived from two numbers. The same click happened for uploads: "the filename is attacker input" turned a random string of security advice into one obvious rule — generate the name yourself.
- **Career link:** An AI product's admin console is exactly this note: a paginated list of runs or datasets, a rich-text editor for prompts or reports, a file upload for documents to embed. The security rules here — sanitize HTML, whitelist uploads, never trust a filename — are the same ones that protect an LLM app from prompt-injection payloads smuggled through rich text or file content. Building these habits now is building the product surface I will ship later.

## 🔗 Related Topics
- [`./04-routing-in-express.md`](./04-routing-in-express.md) — the routes, middleware order, and guards these features mount on.
- [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) — the views, the escaped-vs-raw (`{{ }}` / `{{{ }}}`) decision, and layout reuse.
- [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md) — the `requireAuth` guard protecting the admin upload/editor routes.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the insert/select used by the pagination and create flows.
- [`../databases/07-postgresql-full-text-search.md`](../databases/07-postgresql-full-text-search.md) — the SQL behind the search box wired into this list page.
- [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) — why deep offset pages slow down, and the keyset alternative.
- [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — stored XSS from the editor and upload vectors, in full.
- [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md) — the browser half of every form/upload/fetch here.
- [SUGGESTED] `09-flash-messages-and-form-validation-ux.md` — the reusable pattern for showing validation errors after a redirect (login, create post). Planned in this folder's backlog.
- [SUGGESTED] `bridge-uploads-to-s3-and-cdn.md` — bridge note: replacing disk storage with object storage (S3) and serving via a CDN, the production path from `multer.diskStorage`. Bridges `web_server/` ↔ deployment.
- [SUGGESTED] `10-image-processing-and-thumbnails.md` — resizing/optimizing uploads with `sharp`, and why you should never serve user images at full size. `web_server/`.

## 🤔 Open Questions
- [ ] At what row count does offset pagination stop being acceptable, and how do we migrate to keyset pagination without breaking existing URLs?
- [ ] For uploads, is MIME whitelisting enough, or should we also sniff the file's magic bytes to catch a renamed executable?
- [ ] Should rich text be stored as sanitized HTML or as a structured document (JSON) that is sanitized at render time — which is more future-proof?
- [ ] Where is the right place to enforce the upload size cap — multer's `limits`, the reverse proxy's `client_max_body_size`, or both?
- [ ] How do we wire the full-text search box so it composes with pagination without running two expensive queries that disagree?

---
*Every feature is the same spine — route, guard, parser, validate, model, view. The "advanced" part is only the extra box in the middle.*

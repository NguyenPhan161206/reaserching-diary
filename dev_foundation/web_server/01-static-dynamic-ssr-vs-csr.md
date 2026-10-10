# Static vs Dynamic · SSR vs CSR — Where Does the HTML Come From?

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#SSR`, `#CSR`, `#StaticSite`, `#DynamicSite`, `#Rendering`, `#Express`, `#NextJS`, `#WebServer`
**References:** MDN "Server-side rendering" & "Client-side rendering", web.dev Core Web Vitals (TTFB, FCP, LCP), Next.js docs — "Rendering: Server and Client Components", *High Performance Browser Networking* (Grigorik, ch. 10)
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (client vs server JS), [`../web_basics/01-html-structure-and-semantics.md`](../web_basics/01-html-structure-and-semantics.md) (what HTML is)

---

## 📌 Key Takeaways (TL;DR)
- The exam distinction is about **who produces the HTML and when**: a **static** site serves the *same bytes from disk to everyone*; a **dynamic** site *builds* the HTML per request (and can differ per user).
- **SSR** = the server runs a template engine on each request and sends a **complete, ready-to-display HTML page**. Good first paint, crawlable, no JS required. Cost: server CPU per request, a full page round-trip per navigation.
- **CSR** = the server sends an (almost) empty **HTML shell + a JavaScript bundle**; the browser downloads and runs the JS to *build* the DOM. Fast once loaded, app-like, but a blank first paint and SEO/JS-dependency costs.
- **Hybrid** (Next.js, the Joblink stack) combines them: generate HTML on the server (or at build time) *and* hydrate it into an interactive client app.
- Choosing a renderer is a **business decision**, not a style preference: a public product page wants SSR/static for SEO and speed; an internal dashboard behind a login wants CSR for interactivity.

---

## 🧠 Detailed Notes

### 1. Core Concepts — Static vs Dynamic

Every question in this note reduces to one: **when a browser requests a URL, where do the bytes of HTML come from?**

```
STATIC SITE                              DYNAMIC SITE
───────────                              ────────────
Browser ──GET /about──► Server           Browser ──GET /posts?user=42──► Server
                          │                                           │  run code
                          ▼                                           ▼  (query DB,
                  /var/www/about.html                        build HTML now)
                  (a file, byte-for-byte)                    "<h1>Hi, Nguyên</h1>…"
                          │                                           │
                          ▼                                           ▼
                    same to EVERYONE                       different per request/user
```

| | Static site | Dynamic site |
|---|---|---|
| HTML produced | At **build/deploy time**, saved as a file | At **request time**, by code |
| Who sees same bytes | **Everyone** | Depends on request / user / data |
| Server work per request | Read a file, ship it | Run application logic |
| Examples | Docs site, blog, marketing page | Feed, dashboard, e-commerce cart |

A **static site** is not "a site with no JavaScript"; it is a site whose HTML is **precomputed** — the exact file on disk is what every visitor receives. A **dynamic site** is one where the server *computes* the response, usually because the content depends on something that changes (the database, the logged-in user, the time). The classic *dynamic* trigger is state: the same `/cart` URL must show *your* cart, not mine.

> **The subtlety:** dynamic vs static is about **when the HTML is produced**, not about whether the site *looks* interactive. A static page can still run client-side JS; a dynamic page can still be cached. Keep the two axes separate (see §5 and the confusions table).

### 2. SSR — Server-Side Rendering

**Definition:** the server produces a complete HTML document *per request* and sends it to the browser. This is the model of **this course**: Express + a template engine (Handlebars/EJS/...).

```
        SSR — one navigation
─────────────────────────────────────────────────────────────
Browser                       Express (Node)              Template
   │                               │                        engine
   │  GET /posts                   │                          │
   ├──────────────────────────────►│                          │
   │                               │  query DB / build data   │
   │                               ├─────────────────────────►│
   │                               │   render("posts", data)  │
   │                               │◄─────────────────────────┤
   │   200 OK                     │   "<html><h1>…</h1></html>"
   │◄──────────────────────────────┤  (COMPLETE page)
   │                               │
   │  browser parses → PAINTS      │
   │  (already has all content)    │
```

**What the browser receives is a finished page.** It contains the header, the list of posts, the footer — everything. The browser can paint it *without executing a single line of your JavaScript*. Contrast that with CSR (§3), where the browser receives a shell and has to build the page itself.

The minimal Express shape (the real code lives in the routing/template notes):

```js
// server-side rendering with a template engine (pseudocode of the course pattern)
app.get("/posts", async (req, res) => {
  const posts = await db.query("SELECT * FROM posts ORDER BY created_at DESC");
  res.render("posts", { posts, user: req.session.user }); // → full HTML string
});
```

**Key properties, each with a "why it matters":**

| Property | Mechanism | Why it matters |
|---|---|---|
| **Good TTFB / FCP** | Server sends finished HTML; browser paints immediately | The user sees content on the *first* paint — no blank white flash |
| **SEO-crawlable** | A crawler that does not run JS still sees the full content | Google/Bing can index a public product page correctly |
| **Works without JS** | HTML is already complete; JS is optional enhancement | Accessibility, older devices, failed bundle loads |
| **Server does the rendering** | Template engine runs on Node's CPU per request | Scales with server resources; heavy pages cost CPU, not client battery |
| **Full round-trip per navigation** | Every link click re-requests a whole document | Simple mental model, but each navigation reloads and re-paints |

**The trade-off in one sentence:** SSR pays **server CPU per request** to guarantee that *every* client — human, crawler, or no-JS device — receives a complete, fast-to-paint page.

### 3. CSR — Client-Side Rendering

**Definition:** the server sends a tiny HTML **shell** plus a `<script>` bundle; the browser downloads the JavaScript, runs it, and *constructs* the DOM itself. The classic result is a **Single-Page Application (SPA)** — one HTML document, navigations handled in JS without full reloads.

```
        CSR — first load
─────────────────────────────────────────────────────────────
Browser                          Server
   │  GET /                         │
   ├───────────────────────────────►│
   │  200 OK                        │
   │◄───────────────────────────────┤
   │  <div id="root"></div>         │   ← empty shell, NO content
   │  <script src="app.js">         │
   │                                │
   │  GET /app.js  (hundreds of KB) │
   ├───────────────────────────────►│
   │◄───────────────────────────────┤
   │  run JS → fetch data → build   │
   │  DOM → now the page appears    │
```

**What the browser receives is an empty container.** Until the JS bundle is downloaded, parsed, executed, and its data fetched, the user sees a blank page (or a spinner). After that, the app is *fast*: clicking between views swaps DOM nodes locally, with no full-page round-trip.

```jsx
// CSR sketch: the HTML shell is empty; JS fills it in
document.getElementById("root").innerHTML = `
  <h1>Posts</h1>
  <ul>${(await fetch("/api/posts").then(r => r.json()))
        .map(p => `<li>${p.title}</li>`).join("")}</ul>
`;
```

**Costs of CSR (the part beginners miss):**

| Cost | Mechanism | Consequence |
|---|---|---|
| **Blank first paint** | Content does not exist until JS runs | Bad FCP/LCP; a visible flash of empty page |
| **SEO needs care** | Crawler may not execute or wait for JS | Public pages risk being indexed as empty |
| **Bigger bundles** | Framework + app code shipped to every client | Slow on mobile / poor networks |
| **Hydration complexity** | Server HTML must be matched by client render | Mismatch bugs, extra CPU |
| **More client CPU** | The user's device does the rendering | A weak phone can lag where the server would not |

**The partial-CSR case the syllabus calls "AJAX".** AJAX is not a full SPA. It is *fragment* rendering: a page is server-rendered (SSR), but one region — a comment list, a search result, a "load more" button — is fetched and replaced by JS *after* load:

```
   full page  = SSR (server sent complete HTML)
   one region = CSR (JS fetches JSON and swaps a <div>)
```

So AJAX sits **between** the two poles: the page is not blank (SSR did that), but part of it updates without a reload. That is why "the syllabus's AJAX chapter is a kind of partial CSR" — the *unit* being rendered is a fragment, not the document.

### 4. Hybrid — SSR + Hydration + CSR (Next.js / Joblink)

Real products rarely pick a pure pole. **Next.js** (the stack behind the [`Joblink`](https://github.com/qminh77/joblink) project) renders on the server **and then boots the same UI as a live client app**:

```
HYBRID flow
───────────
1. Server renders React → sends COMPLETE HTML      (fast FCP, SEO)
2. Browser paints the HTML immediately             (no blank screen)
3. JS bundle downloads, React "hydrates":           (interactivity)
   attaches event handlers to the existing DOM
4. After that, navigation is client-side            (SPA feel)
```

- **Hydration** is the bridge: the server's HTML is *reused*, not thrown away; client JS merely wires up interactivity onto it.
- **Static Generation (SSG)** goes one step further: render the HTML *at build time* and serve it as a static file, revalidating on a schedule. A marketing page with no per-user data needs no server rendering at request time at all.

Teams combine the three when one page must satisfy two audiences: a crawler that needs HTML *now* and a user who needs a snappy app *after*. That is precisely the Joblink situation — public job listings (want SSR/SSG) plus an interactive application flow (wants CSR).

### 5. Decision Guidance

**The comparison table (≥6 dimensions):**

| Dimension | Static | SSR | CSR | Hybrid (Next.js) |
|---|---|---|---|---|
| **SEO indexability** | ✅ Excellent | ✅ Excellent | ⚠️ Needs JS-capable crawler | ✅ Excellent |
| **TTFB** | ✅ Fastest (file) | ⚠️ Slower (renders) | ✅ Fast (tiny shell) | ✅ Fast (static/build) |
| **FCP / LCP** | ✅ Instant | ✅ Good | ❌ Blank until JS runs | ✅ Good |
| **Server CPU per request** | ✅ None | ❌ High (renders each time) | ✅ None | ✅ Low (pre-rendered) |
| **Client load / JS weight** | ✅ Minimal | 🟡 Optional | ❌ Large bundle | 🟡 Medium (hydration) |
| **Interactivity after load** | 🟡 Needs JS anyway | 🟡 Full reload per nav | ✅ Best (app-like) | ✅ Best (after hydration) |
| **Complexity** | ✅ Lowest | 🟡 Medium | 🟡 Medium | ❌ Highest (two worlds) |
| **Data freshness** | ❌ Stale until rebuild | ✅ Always fresh | ✅ Fresh (fetched live) | ✅ Fresh (per strategy) |

**Scenario decisions (the part exams and real work care about):**

- **E-commerce landing page / product listing (Business setting)** → **SSR or SSG.** It must rank on search, show content instantly on a cold mobile connection, and the content is the same for most visitors. Static/SSR wins.
- **Internal analytics dashboard behind a login** → **CSR.** SEO is irrelevant (crawlers do not log in), the data changes constantly and per user, and the value is interaction — charts, filters, drill-downs. Payload rebuilt in the browser.
- **Public marketing page with rare updates** → **Static (SSG).** Cheapest and fastest: no server rendering at request time at all.
- **Mixed app: public catalogue + logged-in workspace** → **Hybrid.** SSR/SSG the catalogue for SEO; CSR the workspace for interactivity.
- **Admin CRUD over a form** → the course's **SSR + template engine** is more than enough; adding a SPA would add bundle and hydration complexity for no benefit.

### 6. Why It Matters

1. **It is an exam-offered distinction.** "Compare static and dynamic pages; compare SSR and CSR" is a standard question, and the *right* answer is not "SSR is old, CSR is modern" — it is a trade-off table like §5.
2. **As an AI Engineer, you ship both kinds of surfaces.** Internal model dashboards, experiment monitors, and chat interfaces are **CSR** (logged-in, interactive, data-heavy). Public product pages, documentation, and landing pages are **SSR/static** (crawlable, instant). Picking the wrong one is an avoidable performance or SEO failure.
3. **It is a business decision, not a taste decision.** SEO and first-paint translate directly into traffic and conversion; server CPU translates into hosting cost. Choosing a renderer is choosing which of those to optimise.
4. **The mechanics connect to the browser side you already know.** What the browser *does* with the received HTML — parse, build the DOM, and run the event loop that schedules your JS — is the subject of [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md).

---

## ⚠️ Pitfalls & Confusions

| Confusion | Why it is wrong | Correct mental model |
|---|---|---|
| "SSR is just a cached static page." | A cache is an *optimisation*; SSR *produces* HTML per request | SSR is dynamic rendering; caching may be layered on top |
| "Static site = SSR." | Static HTML is produced at build time; SSR at request time | Both send full HTML, but static never runs your code per request |
| "Dynamic site means JavaScript-heavy / SPA." | "Dynamic" describes content computed per request, not the client tech | A dynamic site can be plain SSR HTML with zero client JS |
| "CSR == SPA only." | AJAX fragment updates are also (partial) CSR | CSR is *any* rendering done in the browser; SPA is the extreme case |
| "Hydration = re-rendering the page in JS." | Hydration *reuses* server HTML and attaches handlers | It does not recreate the DOM; it adopts it |
| "CSR is always faster." | Only *after* the bundle loads; first paint is slower | CSR wins interactivity, loses first paint and bundle size |
| "SSR means no client JS ever." | SSR can still ship JS for enhancement (the hybrid case) | SSR = the *initial* HTML is complete; JS is optional |

---

## 🔗 Tech Cross-References
- **JavaScript (client vs server)** — the same language runs in both renderers: the browser builds the DOM in CSR; Node builds the HTML string in SSR. See [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md).
- **Template engines** — the tool that actually turns `{ posts }` into HTML in the SSR flow (Handlebars/EJS/Layout). See [`../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md).
- **Application architecture models** — SSR/CSR is a *presentation* choice that sits on top of the 3-tier architecture; the layering is the subject of [`../../software_architecture/03-application-architecture-models.md`](../../software_architecture/03-application-architecture-models.md).
- **Browser rendering / event loop** — what the browser does with the bytes it receives, in both models: [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md).

## 📝 Research Journey
- **Why:** I could write HTML and a bit of JS, but I had no idea *where the page actually comes from*. I assumed "the website makes the page" — which is exactly the confusion this note removes. The course names static/dynamic and SSR/CSR in the first chapter, and I wanted to stop memorising the words and start seeing the data flow.
- **Struggle:** I kept conflating two separate axes: *static vs dynamic* (when is the HTML produced?) and *SSR vs CSR* (who produces it — server or browser?). Also, "rendering" sounded like one thing, but SSR renders a *string* on a server and CSR renders a *DOM* in a browser; only after drawing both flows as ASCII diagrams did the difference become physical.
- **Aha moment:** Watching the CSR flow leave the user staring at an **empty `<div id="root">`** until JS arrives. That single blank element is the entire cost of CSR — and the entire value of SSR. From then on, "why is the page blank for a second?" and "why does the crawler see nothing?" became the same question with the same answer.
- **Career link:** As an AI Engineer in Economics/Business, I will build internal dashboards (CSR — interactive, behind a login, no SEO) *and* public, crawlable product pages (SSR/static). Knowing which render to reach for is the difference between a fast, discoverable product and an expensive blank screen.

## 🔗 Related Topics
- [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) — the process that actually receives the request that this note's flows start from.
- [`../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md) — the renderer behind the SSR flow.
- [`../../software_architecture/03-application-architecture-models.md`](../../software_architecture/03-application-architecture-models.md) — where rendering choices sit in a larger system.
- 🔗 [Joblink](https://github.com/qminh77/joblink) — a Next.js (hybrid) implementation of the same ideas; contrast its stack with this course's Express SSR.
- [SUGGESTED] `bridge-ssr-to-nextjs.md` — bridge note: what the classic Express SSR course teaches vs what React/Next.js changed (components, hydration, data-fetching), because the Joblink codebase assumes it.
- [SUGGESTED] `caching-and-cdn-for-rendered-pages.md` — how a real deployment turns expensive SSR into cheap static delivery at the edge.

## 🤔 Open Questions
- [ ] When SSR output is cached (per URL or per user), how do we keep per-user content correct without leaking one user's page to another?
- [ ] What exactly does a JS-executing crawler (Googlebot) wait for before it "sees" CSR content — and how long?
- [ ] At what traffic level does SSR's per-request CPU cost force a team toward static generation or a CDN?
- [ ] Is hydration ever fully avoidable (islands / partial hydration), or is some client cost always paid for interactivity?

---
*The page is not "there." It is *produced* — the only question is by whom, and when.*

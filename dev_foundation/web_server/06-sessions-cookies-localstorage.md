# State Across Stateless HTTP — Sessions, Cookies & LocalStorage

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#HTTP`, `#Cookies`, `#Sessions`, `#LocalStorage`, `#SessionStorage`, `#Auth`, `#CSRF`, `#XSS`, `#Express`
**References:** RFC 6265 (HTTP State Management Mechanism / cookies) · MDN — "HTTP cookies", "Window.localStorage", "Window.sessionStorage", "SameSite cookies explained" (web.dev) · Express `cookie-session` & `express-session` docs · `connect-session-knex` docs · OWASP — "Session Management Cheat Sheet", "Cross-Site Request Forgery Prevention"
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`./01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md) (what a request/response is), [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) (HTTP headers on the wire), [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (JS objects/JSON for client storage)

---

## 📌 Key Takeaways (TL;DR)
- **HTTP is stateless: each request is independent.** The server does not remember your previous request unless *you* carry state forward. State travels in exactly three places: a **cookie** (wire), a **server-side session** referenced by an id (server), or **`localStorage`/`sessionStorage`** (browser).
- A **cookie** is a small `key=value` string the browser stores and **automatically echoes on every matching request**. That automatic echo is its power (state) and its danger (CSRF).
- A **session** is a server-side record; the browser only holds a random **session id** in an `HttpOnly` cookie. Memory stores die on restart and break multiple instances — production needs a **shared/DB store**.
- **`localStorage` is readable by any JavaScript on the page** (no `HttpOnly`). Therefore it must **never** hold auth tokens unless you accept that one XSS = one stolen session.
- **`HttpOnly` + `Secure` + `SameSite` are non-negotiable on the auth cookie.** `HttpOnly` blunts XSS theft; `SameSite` blunts CSRF; `Secure` keeps it off plain HTTP.
- The exam line that matters: **state management *is* authentication.** Whoever controls where "who am I" lives controls the security of the whole app.

---

## 🧠 Detailed Notes

### 1. Core Concepts — HTTP Forgets Everything

Every HTTP request is a fresh, self-contained message. The server has no built-in memory of the previous one. So how does any website know you logged in on the previous click?

```
   REQUEST 1                          REQUEST 2
   GET /login  ──► server logs in     GET /dashboard ──► server: "who are you?"
   ◄── 200                              ◄── 200 (or 401?)
   ... but request 2 carries NOTHING     unless the browser ADDS state ...
```

State must be **carried** by one of three mechanisms, each living in a different place:

```
   WHERE STATE LIVES
   ─────────────────────────────────────────────────────────────
   Browser storage            Wire (each request)        Server storage
   ┌───────────────┐          ┌─────────────────┐        ┌──────────────────┐
   │ localStorage  │          │ Cookie: sid=abc │        │ session store    │
   │ sessionStorage│          │  (HttpOnly)     │───────►│ {abc: {userId}}  │
   └───────────────┘          └─────────────────┘        └──────────────────┘
     client-side                 automatic                 authoritative
     JS-readable                 echo                      the truth
```

| Layer | Object | Who reads it | Who can tamper |
|---|---|---|---|
| **Wire** | `Cookie` / `Set-Cookie` header | browser sends; server reads | client (it is the client's machine) |
| **Server** | session record in a store | server only | nobody, if the id is unguessable |
| **Browser** | `localStorage` / `sessionStorage` | any JS on the origin | any JS on the origin (XSS) |

**The key insight:** a cookie is *untrusted input* — it is just a header the client controls. A session is *trusted state* — it lives on the server, keyed by an id that must be random enough to be unguessable. Never put the actual permission ("role=admin") in a cookie; put an opaque random id, look up the real record server-side.

---

### 2. Cookies — the Automatic Header

A cookie is a small string the server sets, the browser stores, and the browser **automatically attaches to every request to the matching scope**. It is the only client-storage mechanism visible to the *server*.

**The round trip, on the wire:**

```
   RESPONSE 1 (login)                        REQUEST 2 (any later request)
   ──────────────────                        ─────────────────────────────
   HTTP/1.1 200 OK                           GET /dashboard HTTP/1.1
   Set-Cookie: sid=abc123; HttpOnly;         Cookie: sid=abc123
               Secure; SameSite=Lax;         Host: app.example.com
               Max-Age=86400                 (browser added it automatically)
        │                                         │
        ▼                                         ▼
   browser STORES it                         server reads sid → finds session
```

Server code to set one (Express, no library):

```js
res.setHeader("Set-Cookie", [
  "sid=abc123",
  "HttpOnly",              // JavaScript can't read it (blocks XSS theft)
  "Secure",                // HTTPS only
  "SameSite=Lax",          // not sent on most cross-site requests (blocks CSRF)
  "Path=/",
  "Max-Age=86400",         // 1 day
].join("; "));
```

**The attributes, and what each one protects against:**

| Attribute | Values | Protects against | Why it matters |
|---|---|---|---|
| `Expires` / `Max-Age` | date / seconds | — | session cookie (omitted) dies when the browser closes; `Max-Age` gives an explicit lifetime |
| `Path` | URL path | scope leakage | limits which paths receive the cookie |
| `Domain` | host suffix | scope/leakage | default is the exact host; widening sends it to subdomains too — be careful |
| `HttpOnly` | flag | **XSS theft** | browser hides it from `document.cookie`; JS simply cannot read it |
| `Secure` | flag | network sniffing | cookie only sent over HTTPS (bridge `02` TLS / Nginx) |
| `SameSite` | `Strict`/`Lax`/`None` | **CSRF** | when the browser attaches the cookie on cross-site requests |

**`SameSite` in one table** (the one that confuses everyone):

| Value | Sent on cross-site requests? | Use for |
|---|---|---|
| `Strict` | ❌ never | high-security cookies; breaks link-from-email login flows |
| `Lax` | ✅ only on top-level GET navigations | the sane default for the auth cookie |
| `None` | ✅ always — **requires `Secure`** | cross-site embeds (third-party widgets); dangerous otherwise |

**Limits and gotchas that bite in production:**
- **≤ 4 KB per cookie.** Never store a big object; store an id.
- **~180 cookies per site** browser cap — enough, but don't spam them.
- **First-party vs third-party.** A cookie for *your* site is first-party; one set by an embedded third site is third-party and increasingly blocked by browsers.
- **Never store secrets or PII in a cookie.** The user can read it (except `HttpOnly`), it is transmitted on every request, and it may be logged. Cookies are for identifiers, flags, and preferences — not passwords, not tokens you cannot afford to leak.

**Why cookies are a CSRF seed:** because the browser attaches them *automatically*, a request forged by another site ("submit this form to your bank") arrives **with your cookie already on it**. The server cannot tell the forgery from a real click — unless `SameSite` or an anti-CSRF token stops it. Full attack in [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).

---

### 3. Sessions — Server-Side State, Client-Side Key

A **session** is a server-side record — `{ userId, role, createdAt }` — plus a random **session id** that identifies it. The browser stores only `sid=<random>` in an `HttpOnly` cookie. The server looks up the record by id on each request. The client never sees the data and cannot forge it without guessing the id.

```
   SESSION MODEL
   ──────────────
   Browser                          Server
   Cookie: sid=abc123    ─────────►  store.get("abc123")
                                     → { userId: 42, role: "user" }
                                     (the authoritative record)
```

**Why not just put `{userId:42}` in the cookie?** Because the client can edit it. A signed session lets the server *detect* tampering, but the clean design is: **cookie = opaque random id; session data = server memory/DB.** If you must keep data client-side, use a *signed* cookie (`cookie-session`) so tampering is rejected — but that puts the data back on the wire every request and cannot be centrally revoked as easily.

**Configuring `express-session` (the course pattern):**

```js
const session = require("express-session");
const KnexSessionStore = require("connect-session-knex")(session);
const knex = require("./db");

app.use(session({
  secret: process.env.SESSION_SECRET,   // signs the id cookie; keep it secret
  name: "sid",                          // don't use the default name
  resave: false,                        // don't rewrite unchanged sessions
  saveUninitialized: false,             // don't create a session for anonymous users
  store: new KnexSessionStore({ knex }), // DB-backed: survives restarts, shared across instances
  cookie: {
    httpOnly: true,                     // JS can't read the id
    secure: process.env.NODE_ENV === "production", // HTTPS only in prod
    sameSite: "lax",
    maxAge: 1000 * 60 * 60 * 24,        // 1 day
  },
}));

// login: put who they are INTO the session
app.post("/login", async (req, res) => {
  const user = await knex("users").where({ email: req.body.email }).first();
  req.session.regenerate((err) => {        // NEW id — defeats session fixation (see below)
    req.session.userId = user.id;
    res.redirect("/dashboard");
  });
});

// read it back on any protected request
function requireAuth(req, res, next) {
  if (req.session.userId) return next();
  res.redirect("/login");
}

// logout: destroy the server record and clear the cookie
app.post("/logout", (req, res) => {
  req.session.destroy(() => res.clearCookie("sid").redirect("/login"));
});
```

**In-memory store vs DB-backed store — the production trap:**

| Store | Survives restart? | Works with >1 app instance? | When |
|---|---|---|---|
| Memory (`MemoryStore`, the default) | ❌ all logins lost on restart | ❌ instance A can't read instance B's sessions | dev only |
| DB (`connect-session-knex` / Redis) | ✅ | ✅ shared | production |

> The default `MemoryStore` is a **development convenience** that leaks memory and is explicitly not for production. Restart `npm run dev` and every user is logged out — and if you scale to two Node processes behind Nginx, half the requests hit an instance that has never heard of your session. That is a distributed-state problem: the session must live somewhere **shared** (bridge [`../../software_architecture/02-modular-monolith-vs-microservices.md`](../../software_architecture/02-modular-monolith-vs-microservices.md) and Docker deploys).

**Session lifecycle:** create on login → carry an `HttpOnly` id cookie with a TTL → refresh/expire on activity → `destroy()` on logout. Fixing the TTL is a security knob: shorter means less exposure, longer means fewer logouts.

**Session fixation & regeneration.** If the server reuses the same session id across the login boundary, an attacker who can plant a known id in the victim's browser (e.g., via a link `?sid=attackerKnown`) can later ride that id once the victim logs in. The fix is to **regenerate the id on privilege change** — `req.session.regenerate(...)` at login — so the pre-login id becomes worthless. This is the concrete bridge to [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md).

---

### 4. LocalStorage & SessionStorage — Browser-Only Storage

Web Storage is a simple synchronous `key → value` map exposed as a JavaScript API. It never touches the server automatically — if you want it in a request, *you* add it (to a header, body, or query), which is exactly why it is **not** a substitute for a session cookie.

```js
// localStorage — persists across tab close and browser restart (within the origin)
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");     // "dark" or null
localStorage.removeItem("theme");
localStorage.clear();

// objects must be serialized — Web Storage only stores strings
localStorage.setItem("filters", JSON.stringify({ q: "shoes", page: 2 }));
const filters = JSON.parse(localStorage.getItem("filters") || "{}");

// sessionStorage — identical API, but scoped to ONE tab, cleared when it closes
sessionStorage.setItem("draft", "half-written comment");

// the storage event fires in OTHER tabs of the same origin (not the one that wrote)
window.addEventListener("storage", (e) => console.log(e.key, e.newValue));
```

**The two variants:**

| | `localStorage` | `sessionStorage` |
|---|---|---|
| Lifetime | until explicitly cleared / user clears data | until the **tab** closes |
| Scope | all tabs/windows on the origin | one tab only |
| Typical use | theme, language, cached read-models, auth token (⚠️) | per-tab draft, wizard step, scroll position |

**Capacity:** ~**5 MB** per origin — orders of magnitude more than a 4 KB cookie, which is why it is tempting for caching. It is **synchronous**, so writing large blobs can briefly block the main thread (bridge [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md)).

**The security truth: there is no `HttpOnly` for Web Storage.** Any script running on the origin — your code *or an injected one* — can read every value. So:
- ✅ Fine: UI preferences, drafts, UI state, a **non-sensitive** cache of data you have already shown.
- ⚠️ Risky: caching PII you would not want leaked.
- ❌ Bad: storing a **session token / JWT** used for authentication. One XSS and the attacker reads it directly. A session id in an `HttpOnly` cookie is *strictly* safer because script cannot read it.

**The JSON-parse gotcha:** `JSON.parse(null)` throws, and so does `JSON.parse` of any corrupted string. Always default: `JSON.parse(localStorage.getItem(k) || "null")` and wrap in `try/catch` for user-influenced keys.

---

### 5. Choosing — the Decision Table

| Dimension | Cookie | `localStorage` | `sessionStorage` |
|---|---|---|---|
| Readable by server? | ✅ (auto-sent) | ❌ (you must attach it) | ❌ (you must attach it) |
| Readable by JS? | ❌ if `HttpOnly` | ✅ always | ✅ always |
| Capacity | ~4 KB | ~5 MB | ~5 MB |
| Lifetime | `Max-Age`/`Expires` or session | until cleared | until tab closes |
| Scope across tabs | all matching requests | all tabs, same origin | one tab |
| Can be `HttpOnly`? | ✅ yes | ❌ no | ❌ no |
| XSS exposure | low (`HttpOnly`) | **high** | **high** |
| CSRF exposure | **yes** (auto-sent) | no | no |
| Best for | session id, auth flag, prefs the server needs | theme, cached read-models, drafts | per-tab wizard/draft state |

**The decision rule in prose:**
1. Does the **server** need to see it on every request without JavaScript? → **cookie** (and if it identifies a login, make it the session id: `HttpOnly` + `Secure` + `SameSite`).
2. Does **only the client** need it, and should it survive a tab close? → **`localStorage`**.
3. Does only the client need it, for **one tab, one session**? → **`sessionStorage`**.
4. Never put an auth secret anywhere JavaScript can read it. Prefer the `HttpOnly` session cookie.

---

### 6. In the Course Stack — login → cookie → session round trip

The course's authentication uses **server-side sessions with a hardened cookie** and keeps **client preferences in `localStorage`**. The full round trip:

```
   1) LOGIN
   Browser  POST /login {email,password}   ─────────►  Express
                                                       verify password (hash!)
                                                       req.session.regenerate()
                                                       create session {userId:42}
   Browser  ◄── 302 /dashboard + Set-Cookie: sid=<rand>; HttpOnly; Secure; SameSite=Lax

   2) NEXT REQUEST (browser attaches the cookie automatically)
   Browser  GET /dashboard   Cookie: sid=<rand>   ───►  Express
                                                       store.get("sid") → {userId:42}
                                                       render dashboard for user 42
   Browser  ◄── 200 OK + HTML

   3) CLIENT-ONLY STATE (server never sees it)
   Browser  localStorage.setItem("theme","dark")   // stays in the browser
```

```js
// Knex lookups make the session *meaningful* (bridge databases/06)
const user = await knex("users").where({ id: req.session.userId }).first();

// a client preference — no cookie, no server involvement
// (browser side)
localStorage.setItem("sidebar", "collapsed");
```

The division of responsibility is the lesson: **authentication and identity** ride the hardened cookie + server session; **cosmetic/UX preferences** ride `localStorage`. Mixing them — putting the auth token in `localStorage` because it is "easier than cookies" — is the most common way a course project becomes an insecure product.

---

### 7. Security Bridge — what each attack reads

The two attacks are mirror images, and each one targets a different storage location:

```
   XSS (injected script runs in YOUR page)
       │  reads whatever JS can read
       ├──► localStorage ................. ALL values exposed
       ├──► non-HttpOnly cookies ......... document.cookie exposed
       └──► HttpOnly cookies ............. BLOCKED (JS cannot see them)

   CSRF (another site forges a request)
       │  browser AUTO-ATTACHES cookies to that request
       └──► your auth cookie rides along → server can't tell it from a real click
       └──► defense: SameSite + anti-CSRF token
```

- **Against XSS:** `HttpOnly` does not stop the page from being compromised, but it does keep the *session id* out of the attacker's reach in the common case. This is why the auth cookie must be `HttpOnly`, and why the auth token must **not** live in `localStorage`.
- **Against CSRF:** because the browser sends cookies automatically, `SameSite=Lax` (or `Strict`) plus a per-form **anti-CSRF token** is the non-negotiable defense for any state-changing request. `SameSite=None` re-opens the door and therefore requires `Secure` and a token.
- **`Secure`** closes the network-sniffing gap: without it the session id can be read off a plain-HTTP hop (the Nginx/TLS layer from [`./02-web-server-nginx-apache-express.md`](./02-web-server-nginx-apache-express.md) is where this is enforced).

Full attack mechanics and prevention: [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).

---

## 📊 Comparison Tables (quick reference)

**Where does state live?**

| Mechanism | Location | Server-visible | JS-visible | Survives restart | Primary risk |
|---|---|---|---|---|---|
| Session (DB) | server | ✅ via id | ❌ | ✅ with DB store | store must be shared |
| Cookie | browser ↔ wire | ✅ auto-sent | ❌ if `HttpOnly` | per `Max-Age` | CSRF |
| `localStorage` | browser | ❌ | ✅ | ✅ | XSS |
| `sessionStorage` | browser | ❌ | ✅ | ❌ (tab) | XSS |

**Cookie attribute checklist for the auth cookie:**

| Attribute | Required value | Blocks |
|---|---|---|
| `HttpOnly` | present | XSS reading the id in `document.cookie` |
| `Secure` | present (prod) | transmission over plain HTTP |
| `SameSite` | `Lax`/`Strict` | most cross-site request forgery |
| `Max-Age` / `Expires` | explicit | indefinite sessions |
| `Path` / `Domain` | narrow | leakage to other paths/subdomains |

---

## 📝 Research Journey
- **Why:** Every app I had built "worked" for one user at a time because I never had to remember anyone. The moment the course introduced login, I hit the real question: *how can a memoryless protocol know who I am?* I wanted one clean model — cookie vs session vs `localStorage` — instead of three half-remembered tutorials.
- **Struggle:** I conflated the cookie with the session. I thought "the login is in the cookie", so I tried storing the user id in a cookie and trusting it — the security note made me realise that is forging my own identity. The shift was seeing the cookie as an **opaque key** and the session as the **server-side truth**. A close second was `SameSite=None` silently requiring `Secure`; I set `None` and the cookie vanished until I understood why.
- **Aha moment:** Drawing the login flow as `POST /login → Set-Cookie: sid=<random> → next request carries Cookie: sid` and watching Express magically "remember" me. The magic turned into one line of state (a random id) plus one lookup. From then on, "why am I logged out after restart?" had an obvious answer: I was using the in-memory store.
- **Career link:** As an AI Engineer in Economics/Business, every dashboard I ship has to answer "who is asking, and are they allowed?". Model endpoints, usage meters, per-tenant data — all of it is state management plus auth. Knowing exactly what each storage mechanism exposes to XSS and CSRF is the difference between a product that can hold a company's data and one that leaks it on the first injected script.

## 🔗 Related Topics
- [`./07-authentication-signup-signin-signout.md`](./07-authentication-signup-signin-signout.md) — the login/signup flow that creates the session this note manages; password hashing, session regeneration, route guards.
- [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — XSS/CSRF/session-hijacking mechanics behind §7.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the `users` lookup and the DB-backed session store (`connect-session-knex`).
- [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md) — the JS/DOM layer that reads and writes `localStorage`.
- [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md) — the login form that POSTs credentials, and `credentials: "include"` for cookie-bearing fetches.
- [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) — the views that render "logged in as {{user.name}}" from session data.
- [`../../software_architecture/02-modular-monolith-vs-microservices.md`](../../software_architecture/02-modular-monolith-vs-microservices.md) — why in-memory sessions break once you run more than one instance.
- [SUGGESTED] `bridge-oauth-and-jwt-vs-sessions.md` — bridge note: how OAuth/JWT change this cookie-vs-session trade-off, and why stateless tokens trade revocability for scaling.
- [SUGGESTED] `cookie-consent-and-third-party-cookie-deprecation.md` — the legal/browser-policy side of third-party cookies, relevant to any analytics an AI product adds.

## 🤔 Open Questions
- [ ] Signed stateless tokens (JWT) put data in the client and dodge the DB lookup — but how do we revoke a token before it expires?
- [ ] When exactly does `SameSite=Lax` break a legitimate cross-site flow, and is a per-form CSRF token always required on top of it?
- [ ] `sessionStorage` dies with the tab, which users dislike; is there a safe way to persist a draft across tabs without leaking it to XSS?
- [ ] For a multi-tenant AI dashboard, should the tenant id live in the session cookie or be resolved from the user record on every request — and what does each choice cost?

---
*HTTP forgets. If you want it to remember you, you must hand it a key — and you must guard where that key is kept.*

## ✅ Depth Checklist (Template A)
- [x] Explanation + runnable code/pseudocode per concept
- [x] ASCII diagrams for the request/session flow
- [x] Comparison/decision table across cookie, localStorage, sessionStorage
- [x] "Why it matters" + security analysis (XSS/CSRF, `HttpOnly`/`SameSite`)
- [x] Research Journey + career link
- [x] Cross-folder related topics + [SUGGESTED] additions

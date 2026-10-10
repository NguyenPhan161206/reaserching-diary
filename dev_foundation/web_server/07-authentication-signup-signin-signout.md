# Authentication — Signup · Signin · Signout (Session-Based SSR)

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Authentication`, `#Auth`, `#bcrypt`, `#Sessions`, `#Cookie`, `#Express`, `#Security`, `#SSR`
**References:** [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) · [OWASP — Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) · [Express — `express-session`](https://expressjs.com/en/resources/middleware/session.html) · [connect-session-knex](https://github.com/gx0r/connect-session-knex) · [bcrypt (npm)](https://www.npmjs.com/package/bcrypt) · [Knex — `insert` with `returning`](https://knexjs.org/guide/query-builder.html) · [jwt.io — Introduction](https://jwt.io/introduction)
**Status:** Draft
**Difficulty:** Intermediate → Advanced
**Prerequisites:** [`./06-sessions-cookies-localstorage.md`](./06-sessions-cookies-localstorage.md) (cookies, session id, `HttpOnly`), [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) (Knex insert/select), [`./04-routing-in-express.md`](./04-routing-in-express.md) (middleware guards), [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) (forms/views)

---

## 📌 Key Takeaways (TL;DR)
- **Authentication = who you are; authorization = what you may do.** AuthN *feeds* AuthZ: you can only decide "can this user delete the post?" after you have established *which user it is*. [→ `security/02`](../security/02-authorization-and-dos-defense.md)
- **Never store a plaintext password — ever.** Store a bcrypt hash (`await bcrypt.hash(pw, 12)`); verify with `await bcrypt.compare(pw, hash)`. The hash is salted and deliberately slow so a leaked table is not immediately a list of passwords.
- This course uses **session-based auth**: signup inserts a user row, signin verifies the hash and starts a session (a server-side record keyed by a cookie), signout destroys that record. The cookie is a *pointer*, not the identity.
- **`session.regenerate()` right after login is the session-fixation defense.** A new session id after authentication means an id an attacker gave you before login is now worthless. [→ `./06`](./06-sessions-cookies-localstorage.md)
- A guard is just middleware: `requireAuth` calls `next()` for a logged-in user and `redirect("/signin")` for everyone else. Order matters — it must sit *before* the protected route.
- JWT is the alternative, not the upgrade. Sessions win when the server renders pages and you need immediate revocation; JWTs win for stateless APIs and mobile. This project is SSR, so it keeps sessions. [→ comparison table below]

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

Two different questions live inside the word "login", and confusing them is the source of most broken permission bugs:

```
   AUTHENTICATION (authN)              AUTHORIZATION (authZ)
   "Who are you?"                      "What are you allowed to do?"
   ────────────────────                ─────────────────────────────
   email + password  ───────────►      role === "admin" ? ... : 403
   establishes identity                decides on resources
        │                                      ▲
        └──────────── feeds ──────────────────┘
                  (authZ needs authN first)
```

**Why this ordering matters:** you cannot answer "may this user edit post #7?" until you know *which user* they are. So authentication always runs first and leaves a durable answer behind — here, a `userId` in the session. Every authorization check downstream (`post.author_id === req.session.userId`, or `req.session.role === "admin"`) reads that. AuthZ is covered in depth in [`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md); this note builds the thing that feeds it.

**The session-based model this course teaches** is a three-move lifecycle:

```
  SIGNUP                     SIGNIN                        SIGNOUT
  ──────                     ──────                        ───────
  validate input             fetch user by email           destroy session record
  hash password (bcrypt)     bcrypt.compare(pw, hash)      clear the cookie
  INSERT users row           regenerate session id         redirect home
  (optionally auto-login)    store userId in session
                             redirect to protected page
        │                          │                              │
        ▼                          ▼                              ▼
   a row in `users`        a row in `sessions`            the `sessions` row is gone
                           + a `sid` cookie               + cookie removed
```

The crucial and counter-intuitive part is at the end: **signout destroys the session, not the user.** The user row is untouched; only the *proof that this browser was logged in* is thrown away. That is why you can log back in immediately with the same credentials — the account never left.

**Where the state actually lives.** HTTP is stateless: each request arrives with no memory of the last. The session fixes this by moving the memory to the *server* and leaving only an opaque id in the client's cookie:

```
  browser                         Express + PostgreSQL
  ───────                         ────────────────────
  Cookie: sid=abc123   ────────►  look up sessions WHERE sid='abc123'
                                  └► { userId: 42, role: 'user', cookie }
  (id only, no data)              req.session is now populated
```

Everything in this note — signup, signin, `requireAuth`, signout — is about creating, reading, and deleting that server-side record. Read [`./06-sessions-cookies-localstorage.md`](./06-sessions-cookies-localstorage.md) first if the cookie-vs-session distinction is not yet solid.

---

### 2. Signup

Signup is the creation of an identity. Four jobs, in order: **validate → normalize → hash → insert**. Skipping any one is a real vulnerability.

**a) Validate on the server, not just the browser.** Client-side `required` and `type="email"` are UX; anyone can `curl` your endpoint. Re-check format and length server-side. The minimum rules: a plausible email shape, a password policy (length matters more than symbol soup — OWASP now recommends length over complexity), and confirmation.

**b) Normalize before you compare.** `"Bob@Example.com "` and `"bob@example.com"` are the same mailbox. `trim()` and `toLowerCase()` the email so the unique constraint catches duplicates instead of silently creating two accounts.

**c) Hash the password with bcrypt — never store the plaintext.** bcrypt runs the password through a deliberately slow, salted key-derivation function:

```
        password "hunter2"
              │
              ▼
   ┌─────────────────────────┐
   │ bcrypt.hash(pw, cost=12) │   cost = 2^12 rounds (≈250–400 ms on a laptop)
   └─────────────────────────┘
              │
              ▼
   $2b$12$Fh8L...   ← algorithm + cost + 22-char salt + 31-char hash, all in one string
```

- **Salt** — a random per-password value mixed in *before* hashing, stored inside the output. It makes two identical passwords hash differently, which kills rainbow-table lookups and means one cracked hash tells the attacker nothing about other accounts.
- **Work factor (cost)** — the `12` in `bcrypt.hash(pw, 12)`. Each increment **doubles** the time. `10` is fast (~60–100 ms), `12` is the modern default (~250–400 ms), `14` is heavy. Why not `20`? Because a login takes half a second and an attacker with GPUs is slowed by the same factor — but so is *your* signup endpoint. The sweet spot is "slow enough to matter, fast enough not to hurt" — and it should be re-tuned upward as hardware gets faster.
- **Why not SHA-256?** A plain hash is fast — billions of guesses per second on a GPU. bcrypt (like Argon2, scrypt) is designed to be slow and memory-hungry, so even a stolen table is expensive to crack. "Fast" is the *enemy* of password storage.

```js
const bcrypt = require("bcrypt");
const SALT_ROUNDS = 12;                              // cost factor, not salt length
const passwordHash = await bcrypt.hash(password, SALT_ROUNDS);
// passwordHash === "$2b$12$...."  ← salt and cost are embedded; store this whole string
```

**d) Insert and catch the duplicate.** The email column has a `UNIQUE` constraint, so the database — not a possibly-racing `SELECT` — is the authority. In PostgreSQL a violation throws an error with `code === "23505"`; turn that into a friendly 409 message. Using `.returning()` (covered in [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md)) hands you the created row without a second query.

**Why it matters:** the unique constraint + friendly catch is the only duplicate check that survives concurrency. A "SELECT then INSERT" lets two simultaneous signups both pass the check and both insert.

The full runnable flow is assembled in §7.

---

### 3. Signin

Signin proves a claim: "I am the account with this email, and I know its password." Two rules govern how you prove it safely.

**Rule 1 — fetch by email, then compare with bcrypt, never `==`.**

```js
const user = await findByEmail(email);
const ok = user && await bcrypt.compare(password, user.password_hash);
```

`bcrypt.compare` re-derives the hash from the submitted password using the salt and cost stored in `user.password_hash`, and returns a boolean. You cannot compare hashes with `===`: bcrypt salts each hash randomly, so hashing the same password twice gives two different strings. `compare` is the only correct test.

```js
const express = require("express");
const app = express();
const session = require("express-session");
const KnexSessionStore = require("connect-session-knex")(session);
const db = require("./db");

app.use(session({
  name: "sid",
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: new KnexSessionStore({ knex: db }),
  cookie: {
    httpOnly: true,                               // JS cannot read it (XSS mitigation)
    sameSite: "lax",                              // CSRF mitigation
    secure: process.env.NODE_ENV === "production",// HTTPS-only in production
    maxAge: 1000 * 60 * 60 * 24 * 7,              // 7 days
  },
}));

app.use((req, res, next) => {                     // expose user to every template
  res.locals.currentUser = req.session.userId
    ? { id: req.session.userId, role: req.session.role }
    : null;
  next();
});
```

**Rule 2 — failure must not leak which part was wrong, and must not be free to hammer.** Two problems to defend:

- **Enumeration.** Saying "no such email" versus "wrong password" tells an attacker which emails have accounts. Return one generic message: *"Invalid email or password."* For extra rigor, when the user is not found still run a dummy `bcrypt.compare` against a constant hash, so the response time does not reveal existence.
- **Brute force.** With no limit, an attacker tries millions of passwords. Add a counter + delay per account/IP, or a rate limiter (`express-rate-limit`), or exponential backoff. This is the authentication face of the DoS/abuse problem in [`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md).

**On success, three things happen** (order matters):

```js
req.session.regenerate((err) => {                 // 1. NEW session id (fixation defense)
  if (err) return next(err);
  req.session.userId = user.id;                   // 2. store only the id, never the password
  req.session.role = user.role;
  const target = req.session.returnTo || "/profile";
  delete req.session.returnTo;                    // 3. then redirect
  res.redirect(target);
});
```

**Why `regenerate` before storing the user?** *Session fixation*: an attacker tricks a victim into using a session id the attacker already knows. If login merely *reuses* that pre-login id, the attacker is now logged in as the victim. Giving the authenticated session a **fresh id** (and destroying the old record) makes the known id useless — the attack's whole premise collapses. This is the single most important line in the note. [→ `./06`](./06-sessions-cookies-localstorage.md)

**Flash messages** are the one-time message pattern for the redirect ("Invalid email or password"). Two options: the `connect-flash` middleware, or a small hand-rolled `req.session.flash = {...}` that the next request reads and deletes. Because signin *redirects*, you cannot just render the error in the same response — the message has to survive one request, which is exactly what the session is for.

---

### 4. Signout

Signout has one job: **make the proof of identity unusable.**

```js
function signout(req, res, next) {
  req.session.destroy((err) => {                  // delete the server-side session row
    if (err) return next(err);
    res.clearCookie("sid", { path: "/" });        // tell the browser to drop the cookie
    res.redirect("/");
  });
}
```

`destroy()` removes the session record from the store, so even if the browser *kept* sending the old `sid`, the server would find no session and treat the request as anonymous. `clearCookie("sid")` is belt-and-braces: it removes the client copy too. Both matter because either side alone is a leak — a destroyed server record with a live cookie is harmless, but a live server record with a stolen cookie is a full session.

**Why the user still exists:** signout deletes the *session*, not the `users` row. "Logged in" is a property of a browser/session, not of the account. The account is permanent; the proof is disposable. Sign out, sign back in — same row, brand-new session.

> **Token caveat:** in a JWT world there is no server-side record to delete. `destroy` becomes "the client throws the token away", and the token stays valid until it expires unless you add a server-side denylist. That asymmetry is the whole revocation row in the comparison table.

---

### 5. The Session Plumbing

Signup/signin/signout are handlers; the machinery that makes them *persistent and reusable* is four pieces.

**a) A DB-backed session store.** The default `MemoryStore` loses every session on restart and leaks memory — unusable beyond a demo. `connect-session-knex` stores sessions in a Postgres table so they survive restarts and work across multiple server processes:

```js
const session = require("express-session");
const KnexSessionStore = require("connect-session-knex")(session);

app.use(session({
  secret: process.env.SESSION_SECRET,             // signs the cookie; from env, never committed
  store: new KnexSessionStore({ knex: db, tablename: "sessions" }),
  resave: false,                                  // don't rewrite unchanged sessions
  saveUninitialized: false,                       // don't create sessions for anonymous visitors
  cookie: { httpOnly: true, sameSite: "lax", secure: isProd, maxAge: 7 * 24 * 3600 * 1000 },
}));
```

**TTL** is the `maxAge` on the cookie plus the store's own expiry: the session row is deleted after the window regardless of activity. Short TTLs reduce the value of a stolen cookie; long TTLs reduce how often users re-login. A common compromise: a long session with a shorter "idle timeout" that expires on inactivity.

**b) `requireAuth` — the guard.** A guard is ordinary middleware that either lets the chain continue or ends it. No magic:

```js
function requireAuth(req, res, next) {
  if (req.session.userId) return next();          // authenticated → continue
  req.session.returnTo = req.originalUrl;         // remember where they wanted to go
  res.redirect("/signin");                        // anonymous → login page
}
```

Mount it before any protected route. Order is the whole game — the same principle as note 04's middleware ordering:

```js
router.get("/profile", requireAuth, getProfile);                 // protected
router.get("/admin",   requireAuth, requireRole("admin"), getAdmin); // protected + authorized
```

**c) `requireRole` — the authorization stub.** AuthN has told us *who*; this decides *whether*. It reads the role placed in the session at login and is the seam where full RBAC plugs in ([→ `security/02`](../security/02-authorization-and-dos-defense.md)):

```js
function requireRole(role) {
  return (req, res, next) => {
    if (req.session.role === role) return next();
    res.status(403).render("errors/403");         // authenticated but not allowed
  };
}
```

Notice the status-code distinction: anonymous → `302` to `/signin`; authenticated-but-forbidden → `403`. They are different failures and deserve different responses.

**d) `app.locals.currentUser` / `res.locals.currentUser` for templates.** Every view needs to know whether someone is logged in (to show "Sign out" vs "Sign in"). Populate it once, globally:

```js
app.use((req, res, next) => {
  res.locals.currentUser = req.session.userId
    ? { id: req.session.userId, role: req.session.role }
    : null;
  next();
});
```

`res.locals` is per-response and automatically available to the render engine (`{{#if currentUser}}` in Handlebars), so no controller has to pass it. [→ `./05`](./05-template-engines-handlebars-ejs-layout.md)

**Open-redirect safety in `returnTo`:** never redirect to a URL supplied by the request without checking it is a *relative* path. Storing `req.originalUrl` (always relative) instead of a `?next=` query parameter avoids the classic "login then bounce to `https://evil.com`" phishing trick.

---

### 6. JWT — The Alternative, Honestly Compared

A **JWT** (JSON Web Token) is a signed, self-contained token: `header.payload.signature`, where the payload carries claims like `{ sub: 42, role: "user", exp: ... }`. The server *signs* it with a secret; it does not *store* it. On each request the client sends the token and the server verifies the signature and trusts the embedded claims — no database lookup required. That is the whole appeal: **stateless**.

The trap is that "stateless" is also the weakness. Because the server keeps no record, it cannot *revoke* a token before it expires without building... a server-side denylist, which reintroduces state. And "where does the client keep it?" is a genuine dilemma: `localStorage` is readable by any XSS, and a cookie needs exactly the `HttpOnly` + `SameSite` protections that session cookies need. The tunnel: **JWT-in-a-cookie is still a cookie.**

| Criterion | Session (this course) | JWT |
|---|---|---|
| **Where the state lives** | Server (a `sessions` row) | Client (the token) |
| **Server DB lookup per request** | Yes (read session row) | No (verify signature only) |
| **Revocation** | ✅ delete the row — instant | ❌ not until expiry (needs a denylist) |
| **Horizontal scaling** | Shared store (Postgres/Redis) required | Trivially stateless across servers |
| **Data freshness** | Always current (re-read from row) | Stale until token expires (claims frozen at issue) |
| **Best client** | Browser / SSR pages | Mobile apps, service-to-service, microservices |
| **Cookie/XSS/CSRF surface** | Cookie only, but needs `HttpOnly`+`SameSite`+CSRF token | Depends where stored; cookie → same needs as sessions |
| **Logout semantics** | Real: destroy the row | Cosmetic: token lives until `exp` unless denylisted |
| **Secret management** | `SESSION_SECRET` (signs cookie) | Signing key (compromise = forge any identity) |
| **Token size / traffic** | Tiny cookie id | Larger token on every request |

**Why this course stays with sessions:** the app is **server-rendered** (SSR). Every request already hits Express and, for any data, already hits Postgres — so the "one extra lookup" sessions cost is negligible, and the immediate revocation a session gives is worth far more than statelessness. Cookies are the natural browser transport, and `express-session` integrates with the template `currentUser` pattern in one line. JWT would buy scalability this project does not need while costing the revocation it does. That is the honest analysis — not "JWT is modern, sessions are old."

---

### 7. The Attack Checklist

Read this as a review list before shipping any auth code.

| Attack / mistake | What goes wrong | Defense |
|---|---|---|
| **Plaintext passwords** | DB leak = every password exposed | Store bcrypt hashes only; never log the password |
| **Weak bcrypt cost** | Cheap hashes crack fast | Cost ≥ 10, default 12; retune as hardware improves |
| **No rate limit on signin** | Brute force / credential stuffing / DoS | Per-account+IP counter, delay/backoff, `express-rate-limit` [→ `security/02`](../security/02-authorization-and-dos-defense.md) |
| **User enumeration** | "No such email" reveals accounts | One generic error; constant-ish time via dummy compare |
| **Session fixation** | Attacker's pre-login id hijacks the session | `req.session.regenerate()` immediately after successful login |
| **Missing HTTPS / `Secure` cookie** | Cookie stolen over plain HTTP | `cookie.secure = true` in prod; TLS terminated by Nginx [→ `./02`](./02-web-server-nginx-apache-express.md) |
| **`HttpOnly` missing** | XSS reads the session cookie | `cookie.httpOnly = true` |
| **Raw password in logs** | Secrets in log files/APM | Never log `req.body` for auth routes; redact |
| **User id in URLs** | IDOR / tampering; `/users/42/edit` | Check ownership server-side (`post.author_id === req.session.userId`), don't trust the URL |
| **Open redirect after login** | `?next=https://evil.com` phishing | Redirect only to relative, validated paths |

---

### 8. End-to-End Runnable Example (model → controller → routes → views)

A complete, minimal Express + Knex + Handlebars auth flow. Condensed but runnable.

**Migration — the `users` table**

```js
// migrations/001_create_users.js
exports.up = (knex) =>
  knex.schema.createTable("users", (t) => {
    t.increments("id").primary();
    t.string("email").notNullable().unique();      // uniqueness enforced by the DB
    t.string("password_hash").notNullable();        // the bcrypt string, never plaintext
    t.string("role").notNullable().defaultTo("user");
    t.timestamps(true, true);
  });
exports.down = (knex) => knex.schema.dropTable("users");
```

**Model — `models/users.js`**

```js
const bcrypt = require("bcrypt");
const db = require("../db");
const SALT_ROUNDS = 12;

async function createUser(email, password, role = "user") {
  const passwordHash = await bcrypt.hash(password, SALT_ROUNDS);
  const [user] = await db("users")
    .insert({ email: email.trim().toLowerCase(), password_hash: passwordHash, role })
    .returning(["id", "email", "role"]);            // Postgres RETURNING → the new row
  return user;
}

const findByEmail = (email) =>
  db("users").where({ email: email.trim().toLowerCase() }).first();

const findById = (id) =>
  db("users").where({ id }).select("id", "email", "role").first();

module.exports = { createUser, findByEmail, findById };
```

**Controller — `controllers/auth.js`**

```js
const bcrypt = require("bcrypt");
const { createUser, findByEmail, findById } = require("../models/users");

const EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function validateSignup(email, password, confirm) {
  const errors = [];
  if (!EMAIL_RE.test(email || "")) errors.push("Enter a valid email address.");
  if ((password || "").length < 8) errors.push("Password must be at least 8 characters.");
  if (password !== confirm) errors.push("Passwords do not match.");
  return errors;
}

exports.getSignup = (req, res) => res.render("auth/signup", { title: "Sign up" });

exports.postSignup = async (req, res, next) => {
  const { email, password, confirm } = req.body;
  const errors = validateSignup(email, password, confirm);
  if (errors.length) {
    return res.status(400).render("auth/signup", { title: "Sign up", errors, email });
  }
  try {
    const user = await createUser(email, password);
    req.session.regenerate((err) => {               // fresh id, then auto-login
      if (err) return next(err);
      req.session.userId = user.id;
      req.session.role = user.role;
      res.redirect("/profile");
    });
  } catch (err) {
    if (err.code === "23505") {                     // Postgres unique_violation
      return res.status(409).render("auth/signup", {
        title: "Sign up",
        errors: ["That email is already registered."],
        email,
      });
    }
    next(err);
  }
};

exports.getSignin = (req, res) =>
  res.render("auth/signin", { title: "Sign in", flash: req.session.flash, returnTo: req.query.next });
// (read-and-clear flash here; assigned to the view rather than a global)

exports.postSignin = async (req, res, next) => {
  const { email, password } = req.body;
  const user = await findByEmail(email || "");
  const ok = user && (await bcrypt.compare(password || "", user.password_hash));
  if (!ok) {
    req.session.flash = { type: "error", text: "Invalid email or password." };
    return res.status(401).redirect("/signin");
  }
  req.session.regenerate((err) => {                 // fixation defense
    if (err) return next(err);
    req.session.userId = user.id;
    req.session.role = user.role;
    const target = req.session.returnTo || "/profile";
    delete req.session.returnTo;
    res.redirect(target);
  });
};

exports.signout = (req, res, next) => {
  req.session.destroy((err) => {
    if (err) return next(err);
    res.clearCookie("sid", { path: "/" });
    res.redirect("/");
  });
};

exports.getProfile = async (req, res, next) => {
  const user = await findById(req.session.userId);
  res.render("profile", { title: "Profile", user });
};
```

**Routes + guards — `routes/auth.js`**

```js
const router = require("express").Router();
const auth = require("../controllers/auth");

function requireAuth(req, res, next) {
  if (req.session.userId) return next();
  req.session.returnTo = req.originalUrl;
  return res.redirect("/signin");
}
function requireRole(role) {
  return (req, res, next) =>
    req.session.role === role ? next() : res.status(403).render("errors/403");
}

router.get("/signup", auth.getSignup);
router.post("/signup", auth.postSignup);
router.get("/signin", auth.getSignin);
router.post("/signin", auth.postSignin);
router.post("/signout", auth.signout);
router.get("/profile", requireAuth, auth.getProfile);
router.get("/admin", requireAuth, requireRole("admin"), (req, res) => res.render("admin"));

module.exports = router;
```

**Views — `views/auth/signup.hbs`** (Handlebars auto-escapes with `{{ }}`)

```hbs
<h1>Sign up</h1>
{{#if errors}}<ul class="errors">{{#each errors}}<li>{{this}}</li>{{/each}}</ul>{{/if}}
<form method="post" action="/signup">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" value="{{email}}" required>
  <label for="password">Password</label>
  <input type="password" id="password" name="password" minlength="8" required>
  <label for="confirm">Confirm password</label>
  <input type="password" id="confirm" name="confirm" minlength="8" required>
  <button type="submit">Create account</button>
</form>
```

**Layout snippet — `views/layouts/main.hbs`**

```hbs
<header>
  {{#if currentUser}}
    <span>{{currentUser.role}}</span>
    <form method="post" action="/signout"><button>Sign out</button></form>
  {{else}}
    <a href="/signin">Sign in</a> · <a href="/signup">Sign up</a>
  {{/if}}
</header>
{{{body}}}
```

*What to notice:* the whole flow is symmetric — signup and signin both end in `regenerate` + store id + redirect; signout is their inverse. The template never receives a password or a hash; it only ever sees `currentUser = { id, role }`.

> **Lab:** `npm i express express-handlebars knex pg bcrypt express-session connect-session-knex`, run the migration, start the app, then exercise signup → signin → `/profile` (200) → signout → `/profile` (302 to `/signin`). Verify the `users` row has a `$2b$` hash and the `sessions` row disappears after signout.

---

## 🔗 Tech Cross-References (Section K)

- **Auth ↔ Sessions/Cookies** — the session store, `HttpOnly`/`SameSite` cookie and TTL this flow depends on live in `06`. See [`./06-sessions-cookies-localstorage.md`](./06-sessions-cookies-localstorage.md).
- **Auth ↔ Security** — every attack in note `security/01` (XSS steals the cookie, CSRF replays it, fixation plants it before login) targets this exact flow; the four-counter checklist here is the fix. See [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).
- **Auth ↔ Databases** — the `users` row and `sessions` table are Knex queries; bcrypt hashes are DB blobs you never log. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **Auth ↔ Nginx/Deploy** — production adds HTTPS (Secure cookie) and rate limiting at the proxy in front of this app. See [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md) and [`../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md).

## 📝 Research Journey
- **Why:** Every project in the course before this one was "read data, render data". Authentication is the first feature where the *state about the user* is something the app invents and must protect. As someone building toward AI products, I know every Gen-AI SaaS gates its expensive features behind auth — so this is not academic; it is the wall between "demo" and "product".
- **Struggle:** `bcrypt.compare` confused me for a while because I kept thinking "just compare the stored hash to the hash of the input". It only clicked when I understood that bcrypt **salts randomly**, so the same password produces *different* strings every time — there is nothing to compare with `===`, and `compare` re-derives using the salt embedded in the stored hash. The session-fixation fix (`regenerate`) was the second struggle: it looked redundant until I walked through the attack where an attacker plants a session id before login.
- **Aha moment:** Realizing that authN and authZ are *sequential*, not the same thing, and that the session is simply "the durable answer to who" that authZ reads later. That reframing turned a pile of middleware into one clean pipeline: identify once, then every check downstream is just reading `req.session`.
- **Career link:** An AI Engineer shipping a model-backed SaaS will write signup/signin guards, protect rate limits on the expensive inference endpoints, and store API keys per user. This note is that code. The JWT-vs-session table is also interview material — being able to explain *why* a product chose statelessness and what it cost in revocation is exactly the systems thinking these roles test.

## 🔗 Related Topics
- [`./06-sessions-cookies-localstorage.md`](./06-sessions-cookies-localstorage.md) — the cookie/session mechanism this note builds on: session id, `HttpOnly`, `SameSite`, TTL.
- [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — session hijacking, CSRF, and XSS as they attack this exact flow.
- [`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md) — RBAC, RLS, and rate limiting: the authZ half and the brute-force defense.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the `insert().returning()` and unique-constraint handling used in §2 and §8.
- [`./04-routing-in-express.md`](./04-routing-in-express.md) — guards as middleware and why registration order decides who is protected.
- [`./05-template-engines-handlebars-ejs-layout.md`](./05-template-engines-handlebars-ejs-layout.md) — the forms, `res.locals`, and layout that render this flow.
- [`../web_basics/08-forms-fetch-and-multipart-upload.md`](../web_basics/08-forms-fetch-and-multipart-upload.md) — the browser side of the signup/signin forms (encoding, `name`, POST).
- [SUGGESTED] `09-password-reset-and-email-verification.md` — the natural next auth feature: one-time tokens, expiry, and never emailing a password. Belongs in `web_server/`.
- [SUGGESTED] `bridge-auth-to-ai-api-keys.md` — bridge note: extend this session model to per-user API keys and usage-based rate limits for AI endpoints. Bridges `web_server/` ↔ `ai_ml/`.
- [SUGGESTED] `oauth-and-social-login.md` — the "Sign in with Google" flow (OAuth 2.0/OIDC), which changes the identity source but not the session at the end. `web_server/`.

## 🤔 Open Questions
- [ ] Should the work factor be tuned per-request based on measured hash time at startup, or fixed at deploy time?
- [ ] Where is the cleanest place to enforce per-account lockout — the session store, a dedicated `login_attempts` table, or an edge service like Nginx's `limit_req`?
- [ ] For a mobile client of the same app, is it worth running sessions and JWTs side by side, or should the API be JWT-only with short expiry + refresh tokens?
- [ ] How do we safely rotate `SESSION_SECRET` without logging every user out (or do we accept the logout)?
- [ ] Does storing `role` in the session risk staleness (a demoted admin keeps access until re-login), and when does the performance win stop being worth it?

---
*Authentication is a claim; the session is the evidence. Hash the secret, prove the claim, throw the evidence away on signout.*

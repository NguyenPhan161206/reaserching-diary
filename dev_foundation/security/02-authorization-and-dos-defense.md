# Web Security II — Authorization & Denial-of-Service Defense

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Security`, `#OWASP`, `#Authorization`, `#RBAC`, `#ABAC`, `#IDOR`, `#AccessControl`, `#DoS`, `#RateLimiting`, `#Express`
**References:** [OWASP Top 10 — A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) · [OWASP — Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) · [OWASP — Denial of Service Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html) · [OWASP — Insecure Direct Object Reference Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) · [PortSwigger — Access Control Vulnerabilities](https://portswigger.net/web-security/access-control) · [express-rate-limit](https://www.npmjs.com/package/express-rate-limit) · [Node.js — Security Best Practices](https://nodejs.org/en/learn/getting-started/security-best-practices)
**Status:** Draft
**Difficulty:** Intermediate
**Template:** A (Tech Note)
**Prerequisites:** [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md) (authN, `requireAuth` guard), [`../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md) (`req.session.userId`), [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) (slow queries, pooling)

---

## 📌 Key Takeaways (TL;DR)

- **Authentication ≠ Authorization.** AuthN answers *who you are* and must run first; AuthZ answers *what you may do to this exact object* and must run **on the server, for every request, on every action**.
- **Broken Access Control is OWASP #1 for a reason: it is the easiest to test and the easiest to forget.** Hiding a button is decoration; the server is the only enforcement point.
- **The four authorization models — RBAC, ABAC, object ownership, route allow-list — compose.** The most common real bug is not "wrong model", it is a *missing object-ownership check* (IDOR): changing `:id` to someone else's.
- **Fail closed.** Default-deny beats default-allow: new routes start locked and are opened deliberately, not the reverse.
- **A DoS is not a hack, it is resource starvation.** CPU, sockets, DB connections, and memory are finite; an attacker (or a bot) spends your budget so real users cannot.
- **Amplification is the asymmetric threat:** a 50-byte request can trigger a multi-second query. Cap the *amplifier* — query limits, timeouts, payload size — before adding rate limiting.
- **Defence in depth: edge → app → DB → ops.** Rate limiting alone is not a DoS strategy; neither is a WAF alone. Layers cover each other's gaps.

---

## 🧠 Detailed Notes

### 1. Core Concepts — the two axes of a trust model

Every product's security rests on two questions, and the course devotes a chapter to each:

```
   AUTHENTICATION (authN)                AUTHORIZATION (authZ)
   "Who are you?"                        "What may you do?"
   ──────────────────────                ────────────────────────────
   password / bcrypt / session           role / ownership / policy
   establishes identity ───────► feeds ───► decides on a specific resource

   THE RULE: authZ is enforced on the SERVER, on EVERY request.
             The client is untrusted; a hidden button proves nothing.
```

**Why "the client is untrusted" is the whole game.** Any check that lives only in the browser — hiding an admin link, disabling a form field, validating in JavaScript — can be bypassed by opening DevTools or sending `curl` directly. The browser is a convenience layer; the server is the enforcement layer. If the server does not re-check, the check does not exist.

**OWASP A01 — Broken Access Control** is ranked #1 in the 2021 Top 10 because it is simultaneously *the most common* and *the most trivially exploitable*: the tester just changes an id or a role claim and watches what happens. This note treats it as the central topic.

**DoS belongs here because it is the availability axis of the same model.** Authorization protects *confidentiality and integrity* (who sees/changes what); DoS attacks *availability* (can anyone use the system at all). Both are about who is allowed to consume a limited resource — one is a permissions problem, the other a capacity problem.

---

### 2. Authorization models — four shapes, one server

#### 2.1 The four models at a glance

| Model | Decides based on | Typical check | Strength / weakness |
|---|---|---|---|
| **RBAC** (role-based) | the user's role | `requireRole("admin")` | simple, coarse; roles explode in count |
| **ABAC** (attribute/policy) | attributes of user *and* resource *and* context | policy function | fine-grained, flexible; harder to reason about |
| **Object ownership** | the link between user and object | `post.author_id === req.session.userId` | precise; must be attached to **every** object route |
| **Route/method allow-list** | an explicit table of allowed (role, method, path) pairs | middleware table | auditable, fail-closed; verbose |

#### 2.2 RBAC — role guard middleware

RBAC attaches a role to the user and gates whole route groups. This is the direct extension of the `requireAuth` pattern from `web_server/07`:

```js
// requireAuth: logged in?  (authN)  — from web_server/07
function requireAuth(req, res, next) {
  if (!req.session.userId) return res.redirect("/signin");
  next();
}

// requireRole: has the role?  (authZ, RBAC)
function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.session.role)) {
      return res.status(403).render("error", { message: "Forbidden" });
    }
    next();
  };
}

app.get("/admin/users", requireAuth, requireRole("admin"), adminListUsers);
app.post("/admin/users/:id/ban", requireAuth, requireRole("admin"), adminBanUser);
```

**Order matters:** `requireAuth` **before** `requireRole`, because a role check needs an authenticated identity. Swapping them either crashes or, worse, treats a missing identity as a role.

#### 2.3 ABAC — attribute/policy-based

ABAC evaluates a *policy* over attributes of the subject, the resource, and the context:

```js
function canEditPost(user, post, ctx) {
  if (user.role === "admin") return true;
  if (post.author_id === user.id) return true;
  return post.is_shared && post.co_editor_id === user.id;   // attribute-driven
}

app.put("/posts/:id", requireAuth, async (req, res) => {
  const post = await db("posts").where({ id: req.params.id }).first();
  if (!post) return res.sendStatus(404);
  if (!canEditPost(req.session.user, post, req)) return res.sendStatus(403);
  // ... update ...
});
```

ABAC generalises RBAC and ownership: "can I edit this post?" becomes a function of *who I am, what the post is, and the situation*. Powerful, but the policy is now code you must test — an untested policy is an untested security boundary.

#### 2.4 Object ownership and the IDOR failure

The most common real-world bug is **IDOR** (Insecure Direct Object Reference): a route trusts the `:id` from the URL without checking the object belongs to the caller. This is **horizontal privilege escalation** — a normal user acting on *another normal user's* data.

```js
// ❌ VULNERABLE — deletes ANY post whose id is supplied
app.post("/posts/:id/delete", requireAuth, async (req, res) => {
  await db("posts").where({ id: req.params.id }).del();
  res.redirect("/posts");
});
// Attacker changes the id to 42 (someone else's post) → it disappears.
```

```js
// ✅ FIXED — the query is scoped to the owner in the SAME statement
app.post("/posts/:id/delete", requireAuth, async (req, res) => {
  const n = await db("posts")
    .where({ id: req.params.id, author_id: req.session.userId })  // ownership in SQL
    .del();
  if (n === 0) return res.sendStatus(404);   // not ours OR not existing → same answer
  res.redirect("/posts");
});
```

Two subtle wins in the fixed version: the ownership filter is in the `WHERE`, so there is no window between "fetch" and "check" (a **TOCTOU** race), and a zero-row delete returns **404, not 403** — the same response for "not yours" and "does not exist", so the route does not leak which ids exist.

#### 2.5 Vertical vs horizontal escalation

```
   VERTICAL (role escalation)                 HORIZONTAL (data escalation)
   ─────────────────────────                  ────────────────────────────
   user  ──►  pretends to be  ──► admin       user A ──►  acts on  ──► user B's data
   "I am an admin"                            "this is my post" (it is not)
   defences: RBAC/ABAC on every route,        defences: ownership check on EVERY
   server-side role from session,             object route, scoped query, 404 not 403
   never trust a client role claim

   BOTH are "missing function-level / object-level" checks:
   the endpoint exists and authenticates, but does not AUTHORIZE this actor.
```

#### 2.6 Missing-function access

Even when users cannot reach the UI, the *route* may still exist and be reachable directly. A hidden `/admin` link is not control; the endpoint must return **403** (authenticated but not allowed) or **404** (hide existence) on its own.

```bash
# The authorization test — as a NORMAL user, hit every admin action directly:
curl -i -X POST http://localhost:3000/admin/users/7/ban \
     -H "Cookie: connect.sid=<normal-user-session>"
# Expected: 403 or 404. If you see 200 / a redirect to a success page → broken access control.
```

---

### 3. Common authorization failures (a field guide)

| Failure | How it looks | Fix |
|---|---|---|
| **IDOR** | change `:id` to another user's row | ownership filter in the query; 404 on no-row |
| **Mass assignment** | `db('users').update(req.body)` lets `isAdmin:true` through | allow-list the writable fields |
| **Default-allow** | new route is public until someone remembers to protect it | default-deny; explicit allow table |
| **URL-only / UI-only checks** | button hidden, endpoint open | enforce server-side; UI is decoration |
| **Wrong middleware order** | authZ runs before authN, or not at all on a route | `requireAuth` → `requireRole` → handler, registered in order |
| **Trusting client role** | role read from body/JWT claim without verification | read role from the server-side session/verified token |

**Mass assignment in code:**

```js
// ❌ VULNERABLE — any field the client sends is written, including "role"
app.post("/profile", requireAuth, async (req, res) => {
  await db("users").where({ id: req.session.userId }).update(req.body);
});
// POST body {"display_name":"Bob","role":"admin"} → self-promotion.

// ✅ FIXED — whitelist exactly the fields a user may change
const editable = ["display_name", "bio", "avatar_url"];
const patch = Object.fromEntries(
  Object.entries(req.body).filter(([k]) => editable.includes(k))
);
await db("users").where({ id: req.session.userId }).update(patch);
```

**Default-deny in middleware:**

```js
// ✅ default-deny skeleton: everything under /admin is locked unless opened
app.use("/admin", requireAuth, requireRole("admin"));
app.use("/admin/reports", requireRole("analyst"));   // deliberate widening, documented
```

[→ `../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md) for the view layer that must never be the control, and `web_server/08` for the AJAX endpoints that must carry the same checks as their HTML forms.

**Why it matters:** AuthZ bugs do not crash — they silently return the wrong user's data. There is no stack trace to alert you. The only defence is a habit: for every route that touches an object, ask *"whose object, and did I prove it in the query?"*

---

### 4. Denial of Service — the physics of unavailability

#### 4.1 DoS is resource starvation

A service is "up" only if it can answer within a reasonable time. A DoS spends a **finite resource** so that legitimate requests time out:

```
   THE FOUR FINITE RESOURCES
   ┌──────────┬───────────────────────────────────────────────┐
   │ CPU      │ regex backtracking, huge sorts, compression    │
   │ SOCKETS  │ open connections held idle (slowloris)         │
   │ DB CONNs │ pool exhaustion via slow/expensive queries     │
   │ MEMORY   │ body bombs, unbounded result sets, caches      │
   └──────────┴───────────────────────────────────────────────┘
   Spend one to zero → every other request queues behind it.
```

#### 4.2 Amplification — tiny request, expensive work

The asymmetric threat is **amplification**: a small input triggers work thousands of times larger.

```js
// ❌ AMPLIFIER — user controls sort column and a deep join, no limit
app.get("/report", async (req, res) => {
  const rows = await db("orders")
    .join("users", "users.id", "orders.user_id")
    .join("order_items", "order_items.order_id", "orders.id")
    .orderBy(req.query.sort)          // attacker picks an unindexed column
    .select("*");                     // no LIMIT → returns the whole table
  res.json(rows);                     // serialising millions of rows
});
```

A 60-byte request (`?sort=customer_note`) can force an unindexed sort over millions of rows, exhaust the connection pool, and take the site down for everyone. The attack surface grows with `?id[0]=...`-style array params that explode into giant `IN (...)` clauses, range scans, or regex `LIKE` patterns.

#### 4.3 Distributed DoS and slowloris

- **Volumetric DDoS** comes from thousands of hosts (a botnet), so per-IP limits do little; the edge/CDN absorbs it. The application cannot solve this alone.
- **Slowloris** is the opposite: *few* connections, each opened and then dribbled with bytes so the socket never closes. A single laptop can hold thousands of sockets and starve a server limited by concurrent connections. Mitigation: request timeouts and connection limits at the proxy layer. [→ `../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md)
- **Algorithmic / "billion laughs" style bombs:** deeply nested JSON, XML entity expansion, or a hash-collision body crafted so parsing itself is quadratic.
- **ReDoS (regex DoS):** a pattern with catastrophic backtracking run against a crafted string can block the single Node.js event loop. This is why a "simple" user-input regex or a template helper is a DoS vector — one request freezes the whole process.

```
   ReDoS — catastrophic backtracking
   pattern: /^(a+)+$/           input: "aaaaaaaaaaaaaaaaaaaaaaaaX"
   the (a+)+ has exponentially many ways to partition the a's →
   each failing attempt re-explores them → CPU pegged, event loop blocked.
```

**Why it matters:** this is not an exotic attacker; it is also your own bug. An unbounded query or a careless regex takes the site down under *normal* traffic spikes. DoS defence is really "resource budgeting", and you need it whether or not anyone attacks you.

---

### 5. Rate limiting & traffic defense

#### 5.1 The token-bucket mental model

A **token bucket** gives each client a bucket that refills at a fixed rate; each request takes a token. When the bucket is empty, requests are rejected (429). Two numbers describe behaviour:

- **burst** = bucket size (how many at once)
- **sustained rate** = refill per second (how many over time)

That distinction is why the token bucket beats a naive "N per minute": it allows a legitimate short burst (a page firing several AJAX calls) while capping the long-run rate.

```js
const rateLimit = require("express-rate-limit");

// Global gentle limit
app.use(rateLimit({ windowMs: 60_000, max: 300, standardHeaders: true }));

// Aggressive limit on the expensive/auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60_000,
  max: 10,
  message: "Too many attempts, try again later.",
});
app.post("/signin", authLimiter, signinHandler);
app.post("/signup", authLimiter, signupHandler);
```

**Login throttling + lockout** builds on `web_server/07`'s signin: count failed attempts per account *and* per IP, add backoff, and lock temporarily after N failures. Locking per-IP alone lets a distributed attacker spread attempts; locking per-account alone enables a denial-of-service against a known user. Use both.

#### 5.2 Where to enforce — app *and* edge

Application rate limiting is easy to bypass when you run behind a proxy unless you configure the real client IP (`app.set("trust proxy", 1)`), and it costs CPU because every request still reaches Node. The **edge** (Nginx/CDN), by contrast, rejects abusive traffic before it touches your process. Mature setups do both: coarse limits at the proxy, fine-grained per-route limits in the app. [→ `../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md)

**CAPTCHA** is a *weak-signal* gate — useful to slow automated signups, but not a rate limit and not a resource cap; treat it as one tool among several.

**Timeout middleware** is the slowloris mitigation inside Node: cap how long a request may take, and cap body size.

```js
app.use(require("express").json({ limit: "100kb" }));  // body bomb guard
```

---

### 6. Resource defense — cap the amplifier

Rate limiting controls *how often* requests arrive; resource caps control *how much each one costs*. Cap the amplifier first.

| Resource | Control | Where |
|---|---|---|
| Request body | `express.json({ limit })`, `multer` `limits.fileSize` | app middleware |
| Query result | **always** `LIMIT`, keyset pagination | `databases/06`, `08` |
| Query runtime | `statement_timeout` on the session/role | PostgreSQL |
| DB connections | pool `max`, queue timeout | `databases/08` |
| CPU (regex/own code) | safe regexes, avoid catastrophic backtracking, move heavy work off the event loop | app |
| Sockets/time | proxy `client_header_timeout`, keep-alive limits, request timeout | Nginx |

```sql
-- Per-role statement timeout: no single query may run forever
ALTER ROLE app_user SET statement_timeout = '5s';
```

```js
// Knex pool: fail fast instead of queueing forever
const db = require("knex")({ client: "pg", connection, pool: { min: 2, max: 10 } });
```

[→ `../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) — the slow-query side of the same problem: an index or a bounded query is DoS defence by another name.

**Static-asset scraping and caching:** a scraper pulling large static assets burns bandwidth. Serving them from a CDN or with cache headers moves the cost off your origin. A **WAF** filters known-bad request patterns at the edge; a **proxy config** enforces connection and timeout limits. Neither replaces application-level authorization or resource caps — they are one layer.

**Resilience (operations layer):** health checks + a restart policy + replicas turn a crash into a non-event, and horizontal replicas dilute the effect of a per-process attack. [→ `../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md)

**A DoS runbook (the order that actually works):**

```
   1. IDENTIFY the exhausted resource   → is it CPU, sockets, DB, memory?
   2. CAP that resource                 → LIMIT, timeouts, body size, pool size
   3. RATE LIMIT at app AND edge        → per-route + per-IP, token bucket
   4. SCALE / REPLICATE                 → more replicas, autoscale, CDN
   5. MONITOR + ALERT                   → latency, 5xx rate, pool saturation
```

Note the ordering: capping the amplifier comes *before* rate limiting, because a single amplified request can DoS you — rate limiting only reduces the *number* of them.

---

### 7. Defence-in-depth summary

```
   ┌───────────────────────────────────────────────────────────────────┐
   │ EDGE        reverse proxy / CDN / WAF / rate limit / TLS / timeouts │
   │              (rejects abuse before it costs your process)           │
   ├───────────────────────────────────────────────────────────────────┤
   │ APP         input validation · authN → authZ · object ownership ·   │
   │              CSRF token · body-size caps · safe regex · request TTL │
   ├───────────────────────────────────────────────────────────────────┤
   │ DATABASE    least-privileged role · statement_timeout · LIMIT ·     │
   │              pool max · indexes (avoid slow scans)                  │
   ├───────────────────────────────────────────────────────────────────┤
   │ OPS         replicas · autoscaling · health checks · monitoring/    │
   │              alerting · backups                                     │
   └───────────────────────────────────────────────────────────────────┘
   NO LAYER ALONE IS ENOUGH. Each layer assumes the one above it failed.
```

The principle under the diagram: an attacker must not be able to turn a *single* mistake into a full compromise. Because the layers fail independently, a missed authorization check (app) is still bounded by least privilege (DB); a flood (network) is still absorbed by the edge.

#### 7.1 Model comparison recap

| | RBAC | ABAC | Ownership | Route allow-list |
|---|---|---|---|---|
| Input | role | many attributes | user↔object link | explicit (role, method, path) |
| Granularity | coarse | fine | per-object | per-route |
| Failure mode | role explosion | untested policy | **forgotten on one route** | stale table |
| Best for | admin areas | complex sharing rules | user-generated content | auditable route surface |

#### 7.2 Broken access control — test checklist

- [ ] As a normal user, `curl` **every** admin endpoint directly (vertical escalation).
- [ ] Change `:id` in every object route to another user's id (horizontal / IDOR).
- [ ] Send `{"role":"admin"}` / `{"isAdmin":true}` to every profile/update route (mass assignment).
- [ ] Remove the session cookie — does every protected route reject (not crash)?
- [ ] Try each route with a *different role* than intended.
- [ ] Verify a denied request returns 403 (or 404), never 200.
- [ ] Confirm authZ middleware is registered before the handler on *every* protected route.
- [ ] Check that list endpoints scope results to the current user, not just the detail routes.

**Why it matters:** these two axes — access control and availability — are the foundation of every product's trust model. For an AI Engineer shipping features, the recurring bug is letting a `userId` from the request body override the server session; in an LLM/automation agent, the same mistake lets a *prompt* choose whose data the agent acts on. The rule from note `01` carries over exactly: identity comes from the verified server session, never from client-supplied fields. To the exam, this chapter and `01` both appear as applied questions — usually "what is wrong with this route?", which is precisely the IDOR and mass-assignment pattern above.

---

## 🔗 Tech Cross-References

- **Authorization ↔ Authentication (Express)** — `requireRole` is the direct extension of `web_server/07`'s `requireAuth`; ordering `authN → authZ → handler` is the middleware contract. [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md)
- **Authorization ↔ Sessions** — the role and identity authZ reads must come from the server-side session (`req.session`), never from a request body. [`../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md)
- **DoS ↔ Databases** — amplification is a slow query in disguise; `LIMIT`, `statement_timeout`, indexes, and pool sizing are the DB half of DoS defence. [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md)
- **DoS ↔ Web server / proxy** — slowloris and volumetric floods are stopped at the edge with connection limits and timeouts; rate limiting is most effective before the request reaches Node. [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md)

## 📝 Research Journey

- **Why:** After `web_server/07` I could log users in, but "logged in" said nothing about what they were *allowed to do*. Authorization and DoS were the chapter I understood least, because neither is a library call — they are decisions I have to make on every route and every query.
- **Struggle:** My instinct was to check permissions in the view — hide the delete button for other people's posts and call it done. Accepting that the *only* place that matters is the server query took a real rewrite of how I think about routes. The IDOR example (`DELETE /posts/:id` deleting anyone's post) made the gap concrete: the app looked correct and was completely open.
- **Aha moment:** Two of them. First, that putting `author_id` in the `WHERE` clause is both the ownership check *and* the race-condition fix in one statement — the check and the act are atomic. Second, that DoS is just "resource budgeting": once I saw CPU/sockets/DB/memory as a finite budget, rate limiting stopped looking like a magic middleware and started looking like token bookkeeping.
- **Career link:** As an AI Engineer in Economics/Business, I will ship features — often agents that read and act on a user's data. The exact bugs here are the ones that turn such a feature into a breach: an agent that trusts a `userId` from the request body, or an endpoint that lets a prompt choose which account to act on. Building the reflex "identity comes from the session, ownership is proven in the query" now is what keeps an automation feature safe when it starts touching real business data.

## 🔗 Related Topics

- [`./01-web-security-injection-xss-csrf-session-hijacking.md`](./01-web-security-injection-xss-csrf-session-hijacking.md) — the injection/XSS/CSRF/session half of the security chapter.
- [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) — the query-cost side of DoS; slow queries and amplification share a root cause.
- [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md) — the `requireAuth` guard this note builds `requireRole` on top of.
- [SUGGESTED] `03-rate-limiting-and-edge-defense-deep-dive.md` — a full note on Nginx rate limiting, CDNs, and WAF rules, since this note can only sketch the edge layer.
- [SUGGESTED] `../../ai_ml/09-agent-authorization-and-tool-permissions.md` — RBAC/ownership applied to tool-calling agents: which tools, on whose data, under whose session.
- [SUGGESTED] `04-logging-monitoring-and-audit-trails.md` — OWASP A09; the detection half of access control (you cannot alert on the IDOR you never logged).

## 🤔 Open Questions

- [ ] For a small course project, is a per-request ownership check enough, or is PostgreSQL **RLS** (as Joblink uses) worth the setup — and where exactly does the policy get its "current user"?
- [ ] Where is the line between ABAC policy and ordinary business logic? At what complexity do I extract a policy engine instead of writing `canEditPost()`?
- [ ] `express-rate-limit` with `trust proxy` set: which proxy hops does it trust, and how do I avoid spoofed `X-Forwarded-For` inflating or evading the limit?
- [ ] For DoS, what is the right first cap for a student project — body size, query `LIMIT`, or `statement_timeout` — and how do I measure which one I actually needed?
- [ ] If an LLM agent must act as a user, how do I carry the *session identity* into the tool call so a prompt can never select a different user's data?

---
*Access control answers "whose object is this?", DoS answers "how much of my budget does this request spend?". Get both wrong and the product is open or unavailable — usually the first, then the second.*

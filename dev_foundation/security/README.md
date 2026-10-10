# Security — Web Application Security

**Purpose:** The security chapter of the *Web Application Development* course and — more importantly — the mental model every build in this diary needs. A web app is attack surface; these notes organise the threats by *vector* (how the attack travels) and by *defense layer* (where you stop it), so that fixing one vulnerability teaches the shape of the next.

**Template:** **Template A (Tech Note)** for both notes — security is a *system* property, not a tool you run.

> **Track:** this folder belongs to the **web track**, in parallel with `web_server/` (sessions, auth, escaping live next door) and `databases/` (SQL injection, RLS, least privilege). The Joblink project is the real-world reference: layered security = Zod validation → Server Action guards → PostgreSQL RLS.

## Textbook map

| Syllabus item (ch. 5.4) | Covered by |
|-------------------------|------------|
| SQL Injection · XSS · CSRF · Session Hijacking | `01` |
| Denial of Service · Authorization | `02` |

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | Web security: SQL Injection, XSS, CSRF, Session Hijacking | A | How does an attack travel, and which single control stops it? |
| 02 | Authorization (RBAC, RLS) & DoS defense | A | Who is allowed to do what, and how do we stop resource exhaustion? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| Medium | Security headers & HTTPS/TLS deep dive (Helmet, HSTS, certbot) | Deployment readiness for the course project |
| Low | OWASP Top 10 walkthrough against real payloads | The standard reference list, useful for interviews |
| Low | Secure cookies deep dive (SameSite, prefixes, partitioning) | Note 01 depends on cookie semantics from `web_server/06` |

---

## 🔗 Tech Cross-References (Section K)
- **Security ↔ Express** — middleware order *is* a security boundary: `helmet()`, rate limiters, and guards all run as middleware. See [`../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md).
- **Security ↔ Databases** — parameterised queries defeat SQL injection; RLS is authorization at the storage layer. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) and [`../databases/01-relational-model-and-sql-basics.md`](../databases/01-relational-model-and-sql-basics.md).
- **Security ↔ Template engines** — output escaping in Handlebars/EJS is the default XSS defense; raw output (`{{{ }}}`) re-opens the hole. See [`../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md).
- **Security ↔ Nginx/Docker** — the deployment layer adds rate limiting, TLS termination, and isolation. See [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md) and [`../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md).

## 🔗 Related Topics
- [`../web_server/06-sessions-cookies-localstorage.md`](../web_server/06-sessions-cookies-localstorage.md) — the cookie & session semantics every attack here exploits
- [`../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md) — authentication (who you are) feeding into authorization (what you may do)
- [`../databases/README.md`](../databases/README.md) — data integrity and least privilege at the query layer
- [SUGGESTED] `bridge-security-to-ai-ml.md` — bridge note: adversarial ML, prompt injection, and data poisoning as the AI analogue of web injection

---
*Created: 2026-10-10 — Planned templates: A (system property)*
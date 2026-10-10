# Web Server — Express SSR Backend Track

**Purpose:** The server-side JavaScript (Node/Express) layer of the university course *Web Application Development*. HTML/CSS/JS browser mechanics live in [`../web_basics/`](../web_basics/README.md); the queries those pages turn into data live in [`../databases/`](../databases/README.md); **this folder is the glue** — the HTTP server, the MVC application structure, routing, server-side rendering via template engines, and the session/authentication machinery that makes a static mock-up into a real product.

**Template:** **Template A (Tech Note)** for mechanism/architecture notes (01, 03, 06, 07), **Template G (Dev Tool & Command Note)** for hands-on tooling notes (02, 04, 05, 08).

> **Track:** this is the **web track**. The language here is **JavaScript on the server** (Node.js), the SSR pair to the client-side JavaScript in `web_basics/`. The course's reference architecture is a classic **3-tier server-side-rendered monolith**: browser → Nginx/Express → PostgreSQL. If a topic is about *rendering content in a browser*, it belongs in `web_basics/`; if it is about *receiving a request and producing a response*, it belongs here.

## Purpose in the course

| Chapter of the syllabus | Covered by |
|-------------------------|------------|
| 1.2 Static vs dynamic web · 1.3 SSR vs CSR | `01` |
| 1.5 Web servers (Nginx, Apache) + self-built server (Express) | `02` |
| 3.2 MVC architecture · 3.3 Routing · 3.4 Template engines & Layout | `03` `04` `05` |
| 4.3 Session/Cookie/LocalStorage · 4.4 Authentication | `06` `07` |
| 4.5 Pagination, WYSIWYG, file upload (multipart) | `08` |

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | Static vs Dynamic · SSR vs CSR | A | Where does the HTML come from — a file, a server template, or a client script? |
| 02 | Web server mental model: Nginx / Apache / Express | G | What happens between "browser sends request" and "browser draws response"? |
| 03 | MVC architecture in an Express SSR app | A | How do routes, data, and templates stay separate without falling apart? |
| 04 | Routing & middleware in Express | G | How does a URL become a response, and why does order matter? |
| 05 | Template engines: Handlebars & EJS + Layout | G | How does data become HTML, safely and without repetition? |
| 06 | State management: Session, Cookie, LocalStorage | A | Where does server remember who I am across stateless HTTP requests? |
| 07 | Authentication: Signup / Signin / Signout | A | How does a site know *who* you are, and prove it securely? |
| 08 | Advanced features: pagination, WYSIWYG, upload | G | How do the three "feature-adds" of the course project actually work? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| Medium | Flash messages & form validation UX | The forgotten glue of a real SSR auth flow |
| Medium | JWT vs sessions — a real comparison with revocation | Exam question; most teams move to JWT and regret it |
| Low | Testing an Express app (supertest, mocking DB) | Turns the course project into something maintainable |
| Low | WebSocket / SSE for realtime in SSR | Bridges to `iot_aiot/01-rate-limit-and-sse.md` |

---

## 🔗 Tech Cross-References (Section K)
- **JavaScript ↔ Node** — the same language, two runtimes: browser global `window` vs server global `process`; `require` vs `import`. See [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md).
- **Express ↔ Linux** — a server is a *process* listening on a socket; `lsof -i :3000` and `kill` are half of debugging. See [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md).
- **Express ↔ Databases** — controllers call the data layer (Knex) and hand rows to the template. See [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md).
- **Express ↔ Docker** — the app you built here is packaged into an image and deployed with Compose. See [`../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md).
- **Security** — sessions, escaping, and auth guards are security decisions first. See [`../security/README.md`](../security/README.md).

## 🔗 Related Topics
- [`../web_basics/README.md`](../web_basics/README.md) — the client half of the request/response loop
- [`../databases/README.md`](../databases/README.md) — where the state behind these routes lives
- [`../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md) — the deployment target of this entire course
- [Joblink](https://github.com/qminh77/joblink) — a Next.js CSR/hybrid sibling of the same ideas; the SSR-vs-CSR note contrasts the two stacks
- [SUGGESTED] `bridge-ssr-to-nextjs.md` — bridge note: what the classic SSR course teaches vs what Next.js/React changed

---
*Created: 2026-10-10 — Planned templates: A (mechanism) + G (tooling/hands-on)*
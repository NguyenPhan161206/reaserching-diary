# Databases — SQL, PostgreSQL, MySQL, PostgREST

**Purpose:** Understand where state lives and how to query it safely. From relational model → SQL → engine specifics → instant REST API → schema design decisions.

**Template:** **Template G** for hands-on SQL/engine notes (01, 02, 06, 07), **Template A** for comparisons and design decisions (08).

## Why this matters

- Every real product has a database, and schema mistakes are expensive to undo.
- PostgREST removes the "write a backend to return a table" chore — an AI Engineer's productivity multiplier.
- Joblink already uses Supabase (PostgreSQL + Realtime + RLS): this folder explains that stack.

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | Relational model & SQL basics | G | What is a relational table, and how do keys, joins and `SELECT` work? |
| 02 | PostgreSQL core concepts | G | How does the actual engine store, index and transact this data? |
| 06 | Knex query builder & CRUD | G | How do I query PostgreSQL safely from an Express app without writing SQL strings? |
| 07 | PostgreSQL full-text search | G | How do I search *words* (ranked), not just substrings? |
| 08 | DB bottlenecks: indexing, caching & pooling | A | Why is the database the first bottleneck — and what do I measure and fix first? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | PostgREST — database as REST API | Removes an entire layer of boilerplate; huge leverage for small projects |
| Medium | MySQL vs PostgreSQL | Comparison, not advocacy — know when each wins |
| Medium | Schema design & data modeling | Normalisation, indexes, migrations, the decisions that hurt later |
| Medium | Transactions & isolation levels | The course project writes with Knex transactions (note 06); the theory lands next |
| Low | Connection pooling | Comes up with serverless; underwater until then |
| Low | Vector search for AI applications | Bridges `databases/` ↔ `ai_ml/`; a natural bridge note |

---

## 🔗 Tech Cross-References (Section K)
- **SQL ↔ DSA** — a table is a data structure with a *storage cost model*; B-tree indexes are a data structure with logarithmic search
- **Transactions ↻ Git commits** — `BEGIN`/`COMMIT`/`ROLLBACK` is the same "atomic snapshot of changes" idea applied to data
- **PostgREST ↔ JavaScript** — `fetch('/restaurants?select=*')` returns JSON; the web layer's job is rendering it
- **Indexes ↔ query cost** — an index is a trade: faster reads, slower writes, more disk. Same trade-off as `O(1)` lookup vs `O(n)` scan
- **Docker** — `docker run postgres` gives you a disposable database in seconds
- **RLS / auth ↔ security** — row-level security is the authorization layer in Supabase/Joblink
- **MLOps** — feature stores and data pipelines are database design problems

## 🔗 Related Topics
- [`../docker/README.md`](../docker/README.md) — run databases in containers
- [`../yaml_config/03-config-formats-when-to-use-what.md`](../yaml_config/03-config-formats-when-to-use-what.md) — config & connection settings
- [`../web_basics/README.md`](../web_basics/README.md) — the layer that consumes this data
- [`../web_server/README.md`](../web_server/README.md) — the MVC app whose models call these queries
- [`../security/README.md`](../security/README.md) — parameterised queries + RLS are this layer's authorization
- [`../dsa/README.md`](../dsa/README.md) — the same thinking (data structures + cost models) applied to algorithms
- 🔗 [Joblink](https://github.com/qminh77/joblink) — real PostgreSQL/Supabase schema to read against
- [SUGGESTED] `bridge-sql-to-dsa.md` — bridge note: B-trees, hash indexes, and query planners vs. the data structures learned in DSA

---
*Created: 2026-09-28 — Updated: 2026-10-10 — Templates: G (hands-on) + A (comparison/design)*

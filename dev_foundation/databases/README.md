# Databases — SQL, PostgreSQL, MySQL, PostgREST

**Purpose:** Understand where state lives and how to query it safely. From relational model → SQL → engine specifics → instant REST API → schema design decisions.

**Template:** **Template G** for hands-on SQL/engine notes (01, 02, 04), **Template A** for comparisons and design decisions (03, 05).

## Why this matters

- Every real product has a database, and schema mistakes are expensive to undo.
- PostgREST removes the "write a backend to return a table" chore — an AI Engineer's productivity multiplier.
- Joblink already uses Supabase (PostgreSQL + Realtime + RLS): this folder explains that stack.

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| — | *(none yet)* | — | — |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Relational model & SQL basics (tables, keys, joins, `SELECT`) | The vocabulary every other DB note assumes |
| High | PostgreSQL core concepts (types, indexes, transactions, schema) | The engine Joblink actually runs on |
| High | PostgREST — database as REST API | Removes an entire layer of boilerplate; huge leverage for small projects |
| Medium | MySQL vs PostgreSQL | Comparison, not advocacy — know when each wins |
| Medium | Schema design & data modeling | Normalisation, indexes, migrations, the decisions that hurt later |
| Low | Transactions & isolation levels | Revisit when concurrency actually bites |
| Low | Connection pooling | Comes up with serverless; skip until then |
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
- [`../dsa/README.md`](../dsa/README.md) — the same thinking (data structures + cost models) applied to algorithms
- 🔗 [Joblink](https://github.com/qminh77/joblink) — real PostgreSQL/Supabase schema to read against
- [SUGGESTED] `bridge-sql-to-dsa.md` — bridge note: B-trees, hash indexes, and query planners vs. the data structures learned in DSA

---
*Created: 2026-09-28 — Planned templates: G (hands-on) + A (comparison/design)*

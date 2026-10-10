# PostgreSQL Core Concepts — The Engine Behind the SQL

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#SQL`, `#PostgreSQL`, `#Transactions`, `#Indexes`, `#Docker`, `#ACID`
**References:** PostgreSQL 16 documentation (§3 Advanced Features, §13 Concurrency Control, §19 Server Configuration), the official `postgres` Docker image docs, Vivien & Vial *PostgreSQL: Up and Running*
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`01-relational-model-and-sql-basics.md`](01-relational-model-and-sql-basics.md); Docker basics for `docker run postgres` ([`../docker/README.md`](../docker/README.md))

---

## 📌 Key Takeaways (TL;DR)

- **PostgreSQL is a client/server database.** You do not "open a file" — you connect over a socket as a **role**, every query runs inside a **session**, and writes are protected by **ACID transactions**.
- **MVCC lets readers never block writers.** Every write creates a new row *version*; a transaction sees a consistent snapshot, so a long `SELECT` does not freeze the writers.
- **`docker run postgres` gives you a throwaway database in five seconds** — the fastest way to practice everything here without installing anything.
- **An index is a sorted side-structure** (a B-tree) that turns a $\Theta(n)$ full scan into a $\Theta(\log n)$ lookup — at the cost of slower writes and more disk. Verify with `EXPLAIN ANALYZE`; do not guess.
- **`RETURNING` and `ON CONFLICT … DO UPDATE` remove a whole round-trip** from app code (insert-and-get-id; upsert).
- **Four safe defaults prevent most incidents:** `TIMESTAMPTZ` for events, `NUMERIC` for money, `$1` placeholders for input, and a least-privilege role.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 Client/server, not a file

Unlike SQLite (a library that reads a file), PostgreSQL is a **server process** that owns the data, plus a **client** (`psql`, Knex, your app) that talks to it over TCP or a Unix socket.

```
   your app / psql                         postgres server
   ┌──────────────┐   connect as a role   ┌─────────────────────┐
   │ client       │ ────────────────────► │  postmaster (listen)│
   │ (session)    │   SQL over socket     │   ├── backend: PID 1 │ ← your session
   └──────────────┘ ◄──────────────────── │   ├── backend: PID 2 │ ← another session
             result rows / errors          │   └── shared buffers │
                                            └─────────────────────┘
                                                        │
                                                    data files
```

Four ideas fall out of this picture:

1. **A role (user) is the login identity.** This is `POSTGRES_USER` under Docker. "root" is not how PostgreSQL works — privileges are per-role and checked on every object.
2. **A session is one connection.** Temporary tables, `SET` variables, and open transactions live and die with the session. Connection pooling exists because sessions are expensive to create.
3. **`psql` is just a client.** Everything it does, an app does through the same protocol.
4. **The server is the single source of truth.** Concurrent clients cannot corrupt each other because the server serialises conflicting writes via transactions.

#### 1.2 ACID and MVCC in one paragraph each

- **Atomicity** — a transaction's statements all apply or none do. **Consistency** — constraints hold before and after. **Isolation** — concurrent transactions do not see each other's unfinished work. **Durability** — once `COMMIT` returns, the data survives a crash.
- **MVCC (Multi-Version Concurrency Control)** — instead of locking rows that readers need, PostgreSQL keeps multiple **versions** of a row. A reader sees the snapshot as of its statement; a writer creates a new version. This is why **readers never block writers and writers never block readers** — the most important difference from a naïve locking database.

**Why it matters:** the course's checkout flow (create order → charge → decrement stock) is exactly the multi-step write that needs atomicity. Without `BEGIN`/`COMMIT`, a crash between "order created" and "payment recorded" leaves a half-finished state nobody can reconcile.

---

### 2. Running & Connecting

#### 2.1 A throwaway server with Docker

```bash
docker run -d \
  --name pg16 \
  -p 5432:5432 \
  -e POSTGRES_USER=shopuser \
  -e POSTGRES_PASSWORD=shoppass \
  -e POSTGRES_DB=shop \
  postgres:16
```

- `-d` = detached; `--name pg16` so you can `docker logs pg16` / `docker stop pg16`.
- `-p 5432:5432` publishes the container's port to the host (`host:container`).
- The three `-e` variables create the role, password, and initial database **on first start only** — they are ignored if a data volume already exists.
- With no volume mounted, `docker rm -f pg16` throws the whole database away: perfect for practice, **never** for real data.

```bash
docker exec -it pg16 psql -U shopuser -d shop    # a shell straight into the server
```

#### 2.2 Connecting from the host

```bash
psql "postgres://shopuser:shoppass@localhost:5432/shop"
# or the explicit flags
psql -h localhost -p 5432 -U shopuser -d shop
```

The **connection URI** is what app code and cloud providers use:

```
postgres://USER:PASSWORD@HOST:PORT/DATABASE?sslmode=require
```

This one string is what Knex reads, what Docker Compose passes as `DATABASE_URL`, and what you keep in a `.env` file (never in source).

#### 2.3 Why apps do not connect as `postgres`/root

The superuser bypasses every permission check, so a compromised app yields the whole server. The correct pattern is a dedicated role owning only its schema (see §7). Connection settings come from **environment variables**, never hard-coded, so the same image runs in dev and prod (the twelve-factor rule).

---

### 3. Data Types the Course Needs

| Type | Use for | Watch out for |
|------|---------|---------------|
| `INTEGER` / `BIGINT` | Ids, counts, quantities | `SMALLINT` overflows silently at 32 767 |
| `SERIAL` / `BIGSERIAL` | Legacy auto-increment | **Not rollback-safe** — a failed insert still consumes a number |
| `GENERATED … AS IDENTITY` | Modern auto-increment | Preferred over `SERIAL` in new schemas |
| `NUMERIC(p,s)` | Money, exact decimals | Slower; choose the right scale |
| `REAL` / `DOUBLE PRECISION` | Scientific / ML math | **Binary rounding** — never for money |
| `TEXT` | Most strings | Unlimited length, no padding |
| `VARCHAR(n)` | String with a length limit | `n` is an *upper* bound, not storage |
| `CHAR(n)` | Fixed-width codes | Right-pads with spaces — surprising |
| `BOOLEAN` | Flags | `NULL` is a third value |
| `DATE` | A calendar day | No time, no timezone |
| `TIMESTAMP` | Date + time, **no timezone** | The classic trap — see §3.2 |
| `TIMESTAMPTZ` | Date + time, stored as UTC | The safe default for events |
| `UUID` | Distributed / opaque ids | 16 bytes vs 4 for `INTEGER`; random uuid v4 hurts index locality |
| `JSONB` | Flexible / nested data | Query syntax is verbose; hard to constrain |
| `ARRAY` | Small ordered lists | Not a replacement for a join table |

#### 3.1 Money: `NUMERIC` vs `FLOAT`

`0.1 + 0.2` is not `0.3` in binary floating point:

```sql
SELECT 0.1::float8 + 0.2::float8 = 0.3::float8;   -- false!
SELECT 0.1::numeric + 0.2::numeric = 0.3::numeric; -- true
```

Store money as `NUMERIC(12,2)`. Floats are for physics, not for ledgers.

#### 3.2 The timezone trap

`TIMESTAMP` stores "wall-clock time with no zone" — it forgets *where* the moment happened. `TIMESTAMPTZ` stores an instant (normalised to UTC) and renders it in the session's timezone.

```sql
SET TIME ZONE 'Asia/Ho_Chi_Minh';
SELECT now();                                    -- 2026-10-10 15:00+07
SET TIME ZONE 'UTC';
SELECT now();                                    -- 2026-10-10 08:00+00   (same instant)
```

Rule: **store instants as `TIMESTAMPTZ`**, and convert only for display. Store a *local* date that has no zone (a birthday) as `DATE`.

#### 3.3 `JSONB` — when to reach for it

Use `JSONB` when the shape is genuinely open (a webhook payload, per-product attributes) and you do not filter on individual keys heavily. Do **not** use it to avoid learning schema design: a `JSONB` column you constantly query by key is usually a table you refused to create. `JSONB` stores a parsed binary form, is indexable with GIN, and normalises key order/duplicates.

---

### 4. Transactions & ACID

#### 4.1 The three verbs

```sql
BEGIN;
  INSERT INTO orders (customer_id, total) VALUES (1, 100.00) RETURNING id;
  UPDATE products SET stock = stock - 1 WHERE id = 7;
  -- ... anything here can fail ...
COMMIT;      -- make it permanent
-- or, if something went wrong:
ROLLBACK;    -- undo everything since BEGIN
```

A **savepoint** lets you roll back part of a transaction:

```sql
BEGIN;
  INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
  SAVEPOINT after_order;
  UPDATE inventory SET qty = qty - 1 WHERE product_id = 7;  -- suppose this fails
  ROLLBACK TO after_order;   -- keep the order, discard only the failed step
  INSERT INTO failures (reason) VALUES ('inventory step failed');
COMMIT;
```

#### 4.2 Isolation levels

| Level | Dirty read | Non-repeatable read | Phantom read | Default? |
|-------|-----------|---------------------|--------------|----------|
| `READ COMMITTED` | no | **possible** | possible | ✅ PostgreSQL default |
| `REPEATABLE READ` | no | no | no (PG snapshot) | via `BEGIN ISOLATION LEVEL` |
| `SERIALIZABLE` | no | no | no | strongest; may abort with a serialization error |

**The concurrency example that teaches `READ COMMITTED`.** Two sessions, same row:

| Step | Session A | Session B |
|------|-----------|-----------|
| 1 | `BEGIN;` | |
| 2 | `SELECT stock FROM products WHERE id=7;` → **10** | |
| 3 | | `BEGIN; UPDATE products SET stock=9 WHERE id=7; COMMIT;` |
| 4 | `SELECT stock FROM products WHERE id=7;` → **9** | |
| 5 | `UPDATE products SET stock = stock - 1 WHERE id=7;` | |

Under `READ COMMITTED`, the row A sees in step 2 can change before step 4 — that is the *non-repeatable read*. The saving grace is step 5: `UPDATE … SET stock = stock - 1` reads the **latest** committed version at the moment it runs, so it computes `9 - 1 = 8` rather than clobbering with a stale `10 - 1`. That is why the pattern `SET x = x - 1` is safe and `SET x = <value read earlier>` is not.

**Why it matters:** isolation level is the dial between "fast and occasionally surprising" (`READ COMMITTED`) and "correct under any interference, sometimes retried" (`SERIALIZABLE`). The course flow of "check stock → decrement" needs at least the `UPDATE … SET stock = stock - 1` form, or an explicit `SELECT … FOR UPDATE` lock.

---

### 5. Indexes

#### 5.1 What an index actually is

A table's rows live on disk in **heap** order (insertion order). Finding one row by `id` without help means reading every row: a **sequential scan**, $\Theta(n)$. An **index** is a separate, *sorted* structure that maps key values to row locations, giving $\Theta(\log n)$ lookup on a B-tree.

```
   heap (unsorted)                  B-tree index on id
   ┌──────────────┐                 ┌───────────────┐
   │ id=32 ...    │                 │      16       │
   │ id= 7 ...    │                 │   /      \    │
   │ id=15 ...    │  ◄── pointer ── │  8        24  │
   │ id=16 ...    │                 │ / \      / \  │
   │ id= 8 ...    │                 │7  15   16  32 │
   └──────────────┘                 └───────────────┘
```

The B-tree is the DSA data structure from the searching chapter, now living on disk: balanced, $\Theta(\log n)$ search/insert, sorted for range scans (`BETWEEN`, `ORDER BY`).

#### 5.2 Seeing the planner choose

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 1;
```

Before an index:

```
Seq Scan on orders  (cost=0.00..1834.00 rows=6 width=16) (actual time=0.010..12.4 rows=6 loops=1)
  Filter: (customer_id = 1)
  Rows Removed by Filter: 99994
Planning Time: 0.1 ms
Execution Time: 12.5 ms
```

After `CREATE INDEX idx_orders_customer ON orders(customer_id);`:

```
Index Scan using idx_orders_customer on orders  (cost=0.29..8.31 rows=6 width=16) (actual time=0.03..0.04 rows=6 loops=1)
Execution Time: 0.05 ms
```

Read it left to right: the plan node (`Seq Scan` vs `Index Scan`), estimated cost, actual time, and `Rows Removed by Filter` (the waste the index eliminates).

#### 5.3 The trade-off, and the FK rule

| | Index present | Index absent |
|---|---|---|
| Read one/range | fast ($\Theta(\log n)$) | slow ($\Theta(n)$ scan) |
| Write (`INSERT`/`UPDATE`/`DELETE`) | slower (must update every index) | faster |
| Disk | more | less |
| `ORDER BY` on the indexed column | can be served by the index | needs a sort |

Guidance you should apply by default: **add an index on every foreign-key / join column.** PostgreSQL does *not* create one for `REFERENCES`, and an unindexed FK turns every join into a scan. Do not index everything — every index taxes writes.

- **Bridge to DSA:** choosing a B-tree is a *data-structure selection* decision, and `EXPLAIN ANALYZE` is the database's version of empirical complexity testing. [→ `../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md) and [→ `../dsa/05-data-structure-selection-guide.md`](../dsa/05-data-structure-selection-guide.md)

---

### 6. Essential PostgreSQL Behaviours

#### 6.1 `RETURNING` — get the row back from a write

```sql
INSERT INTO customers (name, email)
VALUES ('Ana', 'ana@x.com')
RETURNING id, created_at;
```

```
 id |         created_at
----+----------------------------
  1 | 2026-10-10 14:03:22.114+07
```

Without `RETURNING`, "insert then get the new id" is two round-trips; with it, one. Works on `UPDATE` and `DELETE` too (`DELETE … RETURNING *` for an undo log).

#### 6.2 `ON CONFLICT … DO UPDATE` (upsert)

```sql
INSERT INTO counters (name, value) VALUES ('visits', 1)
ON CONFLICT (name) DO UPDATE SET value = counters.value + EXCLUDED.value
RETURNING name, value;
```

`EXCLUDED` is the row you *tried* to insert. This requires a **unique constraint or index on the conflict target** (`(name)` here); without it, `ON CONFLICT` silently never fires and the query just errors on the real duplicate instead.

#### 6.3 `SERIAL` and sequences

`SERIAL` is shorthand for "create a sequence, make the column a `BIGINT`/`INTEGER` defaulting to `nextval()`". The sequence is a counter **outside** transactional rollback: a failed or rolled-back insert still consumes a number, so ids have gaps. That is normal and harmless — do not treat ids as contiguous. New schemas should prefer `GENERATED ALWAYS AS IDENTITY`.

#### 6.4 Parameterised placeholders at the protocol level

PostgreSQL's wire protocol sends the query text and the values **separately**. `$1` is not string interpolation; the server parses the statement once with a hole and binds the value as typed data.

```sql
SELECT * FROM users WHERE email = $1;   -- safe against SQL injection by construction
```

This is the mechanism Knex uses under the hood, and it is the real reason input can never be reinterpreted as SQL — a preview of the security chapter. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)

---

### 7. Roles & Privileges

A **role** is a login (with `LOGIN`) and/or a permission container. Lexical convention: `CREATE ROLE app_user` then grant only what it needs.

```sql
CREATE ROLE web_app WITH LOGIN PASSWORD 'secret';
GRANT CONNECT ON DATABASE shop TO web_app;
GRANT USAGE ON SCHEMA public TO web_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO web_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO web_app;   -- for nextval()

-- revoke when no longer needed
REVOKE DELETE ON orders FROM web_app;
```

Every object created in a schema is owned by someone; by default, **`PUBLIC`** (everyone) has some rights, historically including `CREATE` on the `public` schema in older PostgreSQL. Revoke broadly, grant narrowly:

```sql
REVOKE ALL ON SCHEMA public FROM PUBLIC;
GRANT USAGE ON SCHEMA public TO web_app;
```

**Least privilege in practice:** the web app gets DML on its tables and nothing else; migrations run under a separate owner role; analytics gets `SELECT` only. This is the authorization layer that Supabase's RLS and the Joblink stack build on. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)

---

### 8. Command / Code Cheat Sheet

| Command | What it does | When to use | Example | Expected output |
|---------|--------------|-------------|---------|-----------------|
| `docker run -d -p 5432:5432 -e … postgres:16` | Start a throwaway server | Practice / CI / local dev | see §2.1 | long container id |
| `psql "postgres://…"` | Connect with a URI | Match app config exactly | `psql "$DATABASE_URL"` | `shop=#` prompt |
| `\d tablename` | Describe columns/indexes/FKs | "What is this table, really?" | `\d orders` | column table + `Indexes:` |
| `BEGIN; … COMMIT;` | Atomic unit of work | Multi-step writes | see §4.1 | `BEGIN` … `COMMIT` |
| `ROLLBACK;` / `ROLLBACK TO savepoint;` | Undo | After an error mid-transaction | `ROLLBACK;` | `ROLLBACK` |
| `CREATE INDEX idx ON t(col);` | Build a B-tree | Slow lookup on a large table | `CREATE INDEX ON orders(customer_id);` | `CREATE INDEX` |
| `EXPLAIN ANALYZE …` | Show + run the plan | Before/after any index change | `EXPLAIN ANALYZE SELECT …` | `Seq Scan`/`Index Scan` + timings |
| `INSERT … RETURNING …` | Write and read back in one call | Need the new id/timestamp | see §6.1 | the inserted row |
| `INSERT … ON CONFLICT (k) DO UPDATE …` | Upsert | Idempotent "insert or update" | see §6.2 | updated row |
| `SHOW TIME ZONE;` / `SET TIME ZONE 'UTC';` | Inspect / set session zone | Debug timestamp confusion | `SET TIME ZONE 'UTC';` | `SET` |
| `GRANT … / REVOKE …` | Change privileges | Provisioning a role | see §7 | `GRANT` |

---

### 9. Practical Lab

#### Lab 1 — Spin up PostgreSQL, write with `RETURNING`, watch the sequence

```bash
# Step 1: start a throwaway server (see §2.1)
docker run -d --name pg16 -p 5432:5432 \
  -e POSTGRES_USER=shopuser -e POSTGRES_PASSWORD=shoppass -e POSTGRES_DB=shop \
  postgres:16

# Step 2: wire up a shell
docker exec -it pg16 psql -U shopuser -d shop
```

```sql
-- Step 3: a table with an identity column
CREATE TABLE items (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Step 4: insert and read the generated id back in one statement
INSERT INTO items (name) VALUES ('Keyboard') RETURNING id, created_at;
```

**Expected output:**

```
 id |          created_at
----+-------------------------------
  1 | 2026-10-10 07:12:44.982341+00
(1 row)
```

```sql
-- Step 5: prove the sequence keeps moving even when the row is rolled back
BEGIN;
  INSERT INTO items (name) VALUES ('Ghost') RETURNING id;   -- id = 2
ROLLBACK;
INSERT INTO items (name) VALUES ('Mouse') RETURNING id;     -- id = 3, NOT 2
```

**Expected output:** the second insert returns `id = 3`. This is the `SERIAL`/identity gap: rolled-back inserts still consume sequence numbers. Ids are unique, not contiguous.

```bash
# Step 6: tear it all down (this destroys the container's data)
docker rm -f pg16
```

#### Lab 2 — Two sessions, `READ COMMITTED` vs an open transaction

Open **two** terminals, both connected with `docker exec -it pg16 psql -U shopuser -d shop`.

```sql
-- One-time setup (run in Session A)
CREATE TABLE stock (id INT PRIMARY KEY, qty INT NOT NULL);
INSERT INTO stock VALUES (7, 10);
```

**Session A:**

```sql
BEGIN;                                  -- snapshot for this transaction begins
SELECT qty FROM stock WHERE id = 7;     -- first read → 10
```

**Session B (separate terminal):**

```sql
UPDATE stock SET qty = 9 WHERE id = 7;  -- commits immediately (autocommit)
```

**Session A again:**

```sql
SELECT qty FROM stock WHERE id = 7;     -- second read → 9  (non-repeatable read!)
UPDATE stock SET qty = qty - 1 WHERE id = 7;  -- computes 9 - 1 = 8
SELECT qty FROM stock WHERE id = 7;     -- → 8
COMMIT;
```

**What to observe:** under the default `READ COMMITTED`, two `SELECT`s of the *same row inside one transaction* returned **different** values (10, then 9). The `UPDATE … SET qty = qty - 1` was still correct because PostgreSQL re-reads the latest committed version at that moment — the lesson that "read a value, then write `value - 1`" is unsafe, while `SET qty = qty - 1` is safe.

**Then repeat with `BEGIN ISOLATION LEVEL REPEATABLE READ;`** and verify the second `SELECT` still returns `10` (the snapshot is frozen); the writer from Session B is not blocked. That is MVCC in one experiment.

---

### 10. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Using `TIMESTAMP` for events | Times shift by hours after a timezone/deploy change | Use `TIMESTAMPTZ`; convert only for display |
| `SERIAL` treated as contiguous ids | Missing numbers / gaps that "look like deleted rows" | Sequences are not rollback-safe — gaps are normal |
| Missing index on an FK/join column | Join is slow; `EXPLAIN` shows `Seq Scan … Rows Removed by Filter` | `CREATE INDEX` on the referencing column |
| `ON CONFLICT … DO UPDATE` with no matching unique index | Either an error, or it never fires on the intended duplicate | Add a `UNIQUE` constraint/`UNIQUE INDEX` on the conflict target |
| Connection pool exhaustion | App hangs, `too many clients already`, pool timeout | Cap pool size; always release connections; use pooling (e.g. pgBouncer) |
| `float`/`double` for money | Totals off by cents after many operations | `NUMERIC(p,s)` |
| `UPDATE` without up-to-date snapshot semantics | Lost update: two buyers get the last item | `UPDATE … SET qty = qty - 1` (or `SELECT … FOR UPDATE`) |
| Connecting as superuser from the app | One bug = full-server compromise | Least-privilege role, `REVOKE ALL ON SCHEMA public FROM PUBLIC` |
| Hard-coded credentials | Secret leaks into git/`docker inspect`/logs | Env vars / `.env` (git-ignored); never commit secrets |
| Assuming a fresh `docker run` re-runs init | Changed `POSTGRES_PASSWORD` did nothing | Init vars apply only on an empty data dir; remove the volume |
| `VACUUM`/bloat ignored | Table grows although rows are deleted | MVCC keeps dead versions; autovacuum reclaims them (out of scope here, but know it exists) |

---

### 11. Comparison Table — Choosing a Type by Intent

| Intent | Right choice | Wrong choice | Why |
|--------|--------------|--------------|-----|
| Primary key, single node | `INTEGER GENERATED AS IDENTITY` | random `UUID` | sequential ids keep the index compact; random UUIDs scatter writes |
| Money / exact totals | `NUMERIC(12,2)` | `DOUBLE PRECISION` | binary float cannot represent `0.10` exactly |
| "When did this happen?" | `TIMESTAMPTZ` | `TIMESTAMP` | an instant needs a zone; wall-clock does not |
| A birthday / calendar day | `DATE` | `TIMESTAMPTZ` | no time-of-day, no zone |
| Free-form profile text | `TEXT` | `VARCHAR(255)` reflex | `TEXT` + a `CHECK (length(...))` is clearer; no performance difference in PG |
| Open, nested attribute bag | `JSONB` | a dozen nullable columns | indexable (GIN) and schema-flexible |
| Genuinely relational list | a join table | `INTEGER[]` | arrays cannot carry FK integrity or be joined cleanly |
| Idempotent upsert key | `UNIQUE` column | `CHECK` | `ON CONFLICT` needs a *unique* constraint/index |

---

## 🧪 Practice Tasks

> No solutions provided. Do them, then ask for a graded review, an "explain the plan" check, or a harder variant.

- [ ] **Task 1 — Provision by hand.** From a clean `docker run postgres`, create a `bookstore` database, a `web_app` role, and grant it only `SELECT, INSERT, UPDATE` on one table. Prove with `\dp` (or `has_table_privilege`) that it cannot `DELETE`, then grant the missing right.
- [ ] **Task 2 — Prove the index trade-off.** Insert 100 000 rows into a table. Time a lookup, run `EXPLAIN ANALYZE`, add the index, run it again. Then time 100 000 inserts with and without the index and report both deltas.
- [ ] **Task 3 — UUID vs identity.** Create two tables, one with `GENERATED AS IDENTITY` and one with `gen_random_uuid()`. Insert 50 000 rows into each and compare index size (`pg_relation_size`) and insert time. Explain the difference in terms of index locality.
- [ ] **Task 4 — Upsert, correctly.** Build a `page_views(url UNIQUE, views INT)` table and write an upsert that increments on conflict. Then deliberately drop the unique constraint and describe the exact error or silent wrong behaviour you get.
- [ ] **Task 5 (harder) — Reproduce a serialization failure.** With `BEGIN ISOLATION LEVEL SERIALIZABLE` in two sessions, make them conflict and capture the `could not serialize access due to concurrent update` error. Write the retry loop (pseudocode) that a real app would need, and state which isolation level would *not* require it.

## 🔗 Tech Cross-References

- **Docker — the fastest PostgreSQL you will ever own.** `docker run postgres` and `docker compose` with a `DATABASE_URL` give you an isolated engine per project; the compose file that wires the app to the DB is the deploy note. [→ `../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md)
- **DSA — a B-tree index *is* the data structure.** `CREATE INDEX` builds the balanced tree from the searching chapter; `EXPLAIN ANALYZE` is empirical complexity testing on disk. [→ `../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md), [→ `../dsa/05-data-structure-selection-guide.md`](../dsa/05-data-structure-selection-guide.md)
- **Security — `$1` placeholders are the injection defence.** The protocol-level separation of query and value is why parameterised queries cannot be injected; roles/`GRANT` are the authorization half. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)
- **Web server & auth — sessions live in this database.** Signup/signin stores users and sessions as rows, and needs `RETURNING` + unique constraints for correctness. [→ `../web_server/07-authentication-signup-signin-signout.md`](../web_server/07-authentication-signup-signin-signout.md)
- **Knex — the query builder over this engine.** Every Knex call compiles to the SQL here; transactions, `RETURNING`, and `$1` binding are all exposed through it. [→ `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)

---

## 📝 Research Journey

- **Why:** I had used SQL through an ORM and thought I "knew databases" because my queries worked. Then I hit a checkout bug where stock went negative under load, and I realised I had no model of what happened *between* two statements — sessions, snapshots, locks. This note is me learning the engine, not just the query language: what a connection is, what a transaction guarantees, and where a B-tree actually lives.
- **Struggle:** The `READ COMMITTED` example in Lab 2 is what finally cracked it. I had assumed a transaction was a wall around my reads, so seeing the *same* row return `10` and then `9` inside one `BEGIN` felt like a bug. Understanding that PostgreSQL re-evaluates an `UPDATE` against the latest committed row — while a plain `SELECT` does not — separated "reading data" from "mutating data" in my head for good.
- **Aha moment:** Running `EXPLAIN ANALYZE` before and after `CREATE INDEX` and watching `Rows Removed by Filter: 99994` collapse to zero. The cost model from DSA was suddenly *visible on disk*: the index is a B-tree, the heap is the unsorted array, and the planner is choosing the algorithm for me. MVCC was the second one — readers and writers on different *versions* of a row is the whole reason a long report does not freeze checkout.
- **Career link:** Every backend I will build for an economics/business product — orders, ledgers, feature stores — is a set of ACID transactions over a schema. The AI-side work is downstream of this: features are only trustworthy if the transactions that produced them are atomic, and a "training data leakage" bug is often really a transaction/isolation bug. Knowing `RETURNING`, upserts, FK indexes, and least-privilege roles is the difference between a demo and something I would put in front of a business.

## 🔗 Related Topics

- [`01-relational-model-and-sql-basics.md`](01-relational-model-and-sql-basics.md) — the set-based mental model and the SQL this engine runs
- [`README.md`](README.md) — folder roadmap (relational model → engine → PostgREST → schema design)
- [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) — the course's query builder, which emits everything in this note
- [`07-postgresql-full-text-search.md`](07-postgresql-full-text-search.md) — GIN indexes and text search on top of the same index machinery
- [`08-db-bottlenecks-indexing-and-caching.md`](08-db-bottlenecks-indexing-and-caching.md) — where indexes stop being enough
- [SUGGESTED] `09-connection-pooling-and-pgbouncer.md` — the pool-exhaustion pitfall above deserves its own note; serverless + Postgres makes pooling a first-class design decision, and it bridges directly to Knex's pool config.
- [SUGGESTED] `10-migrations-and-schema-evolution.md` — this note creates tables by hand; real projects evolve schema with versioned migrations (Knex migrations), forward-only, reviewable, and reversible. It is the missing bridge from "I can build a schema" to "I can safely change one in production".

## 🤔 Open Questions

- [ ] For the course's checkout flow, is `READ COMMITTED` + `UPDATE … SET stock = stock - 1` genuinely sufficient, or do we need `SELECT … FOR UPDATE` / `SERIALIZABLE` to prevent overselling across concurrent transactions?
- [ ] When should a `UUID` primary key be preferred despite the index-locality cost — and does UUIDv7 (time-ordered) remove the objection?
- [ ] How do I *measure* the point at which adding another index hurts more (write amplification, disk) than it helps (read latency) for a workload like Joblink's?
- [ ] What exactly does autovacuum do under MVCC, and how would I notice table bloat before it becomes a performance incident rather than after?
- [ ] Where is the boundary between "give the app DML on its tables" and "use PostgREST + RLS"? Which layer should own authorization in a small project?

---
*The query language is the vocabulary. The engine — sessions, snapshots, and B-trees — is the grammar that makes it true.*



# Database Bottlenecks, Indexing & Caching - Finding the Query That Eats the Page

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Databases`, `#Performance`, `#PostgreSQL`, `#Indexing`, `#Caching`, `#EXPLAIN`, `#Knex`, `#Bottlenecks`
**References:** PostgreSQL 16 documentation - "Using EXPLAIN" (14.1), "Performance Tips" (14), "Indexes" (11), "Monitoring Database Activity" (27); `postgresql.org`; Martin Kleppmann, *Designing Data-Intensive Applications* (O'Reilly); [martinfowler.com - CacheAside](https://martinfowler.com/bliki/CacheAside.html); the course syllabus 5.3 (database bottlenecks)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) (indexes, `EXPLAIN`, MVCC); [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) (how the app emits queries and owns a connection pool)

---

## Key Takeaways (TL;DR)

- **Performance is mostly architecture plus the database, not clever code.** Every layer (browser -> network -> app -> DB) owns part of the latency, and in an SSR app the database is usually the first to collapse because *every* page funnels through it.
- **Measure before you touch anything.** `EXPLAIN ANALYZE`, `\timing on`, the slow-query log, and a `Server-Timing` header tell you which layer owns which millisecond. Then change **one** thing and re-measure.
- **Six anti-patterns cause most DB pain:** a missing index, N+1 queries, `SELECT *` over fat rows, deep `OFFSET` pagination, wrapping an indexed column in a function, and unbounded scans on huge tables.
- **An index is a data structure with a cost model.** Composite order follows the leftmost-prefix rule; a covering or partial index can save a heap fetch - but every index taxes writes and disk, so an index can *hurt*.
- **Cache-aside is the app-level fix, but invalidation is the hard part.** Know the layer (browser, proxy, Redis, DB), its scope, and who deletes a stale entry - if you cannot answer that, do not cache it.
- **Connection pooling is where correctness and scale meet.** A pool that is too small queues; too large exhausts Postgres; and `max ~ cores x 2 + disks` is the starting point, not a law.

---

## Detailed Notes

### 1. Core Concepts - Where Latency Actually Hides

The course claims "performance is mostly architecture plus the database." Unpack that claim because it is the whole note. In a server-side-rendered (SSR) app, a single page view travels through fixed layers, and each adds time:

$$T_{\text{page}} = T_{\text{browser}} + T_{\text{network}} + T_{\text{app server}} + T_{\text{database}}$$

| Layer | What it owns | Typical cost | Who can fix it |
|-------|--------------|--------------|----------------|
| Browser | parse HTML, fetch CSS/JS, paint | 100500 ms | front-end, `Cache-Control` |
| Network | DNS, TCP, TLS, bytes on the wire | 50300 ms | CDN, gzip/brotli, keep-alive |
| App server | middleware, routing, template render | 5250 ms | view cache, kill sync I/O |
| **Database** | query plan, disk I/O, row transfer | **1 ms  2 s** | indexes, caching, schema |

The DB is the usual first bottleneck for one structural reason: **the app is stateless and cheap, but the database is stateful and shared.** Every request, every user, every concurrent tab funnels into the same engine. Ten app workers scale horizontally by adding processes; they all still hammer one Postgres. When people say "the database is the bottleneck," they mean the fan-in is unavoidable.

Where a database millisecond comes from - this is the mental model for everything below:

```
 a SELECT arrives
       
       
 [1] PARSE + PLAN ......... the planner estimates rows and picks an algorithm
       
       
 [2] FETCH ................. Seq Scan (read every row)  OR  Index Scan (seek a few)
       
       
 [3] JOIN / SORT / AGGREGATE  work proportional to rows touched
       
       
 [4] TRANSFER .............. rows serialised over the wire to the app
```

The two big multipliers are **(2) how many rows the engine touches** and **(4) how many rows and columns cross the wire**. Most fixes attack one of those.

> **Why it matters:** An AI Engineer building an economics/business product will own data systems - RAG document stores, feature stores, event logs. They are all databases with the same fan-in problem. Learning to find the bottleneck query is the difference between "add a bigger server" (guessing) and "this exact query reads 94 000 rows to return 6" (measuring).

---

### 2. Measuring Before Fixing - The Toolbox

Never optimise by intuition. Four tools, in order of usefulness.

#### 2.1 `EXPLAIN ANALYZE` - read the plan, not the number

`EXPLAIN ANALYZE` *runs* the query and prints both the planner's estimate and the real execution.

```sql
EXPLAIN ANALYZE
SELECT id, total FROM orders WHERE customer_id = 42;
```

```
Seq Scan on orders  (cost=0.00..1834.00 rows=6 width=16)
                    (actual time=0.010..12.400 rows=6 loops=1)
  Filter: (customer_id = 42)
  Rows Removed by Filter: 99994
Planning Time: 0.098 ms
Execution Time: 12.512 ms
```

Read it in this order:

1. **Node type** - `Seq Scan` means "read the whole table." `Index Scan` means "seek." This is the first thing to look at.
2. **`rows=` estimate vs `rows=` actual** - when the planner's guess is off by orders of magnitude (a classic cause: stale statistics after a bulk load), it picks the wrong algorithm. `ANALYZE orders;` refreshes statistics.
3. **`Rows Removed by Filter`** - the waste. 99 994 rows read to return 6 is the fingerprint of a missing index.
4. **`actual time`** - real milliseconds, the number users feel.

#### 2.2 `\timing on` - cheap per-statement timing

Inside `psql`:

```sql
\timing on
SELECT count(*) FROM orders;
-- Time: 45.213 ms
```

Fast, but it measures a cold query with no app, no connection overhead. Use it for smoke tests, not conclusions.

#### 2.3 The slow-query log - find the query you forgot

The query you *think* is slow is rarely the one that is. Postgres can log anything over a threshold:

```sql
ALTER SYSTEM SET log_min_duration_statement = '100ms';
SELECT pg_reload_conf();
```

Now every query slower than 100 ms lands in the log with its duration and, if `log_line_prefix` includes `%r`, the client that sent it. This is how you find the N+1 that only triggers on the "orders" page.

#### 2.4 A `Server-Timing` header - measure from the browser

Send timing as an HTTP response header so DevTools' Network panel shows it on the waterfall:

```js
// timing middleware, before your routes
app.use((req, res, next) => {
  const t0 = process.hrtime.bigint();
  res.on('finish', () => {
    const ms = Number(process.hrtime.bigint() - t0) / 1e6;
    res.set('Server-Timing', `app;dur=${ms.toFixed(1)}`);
  });
  next();
});
```

`Server-Timing: app;dur=912.4` next to a 1.8 s page immediately says "half the time is server-side; go look at the query."

#### 2.5 The method - one change at a time

```
 1. Write the query.                          (reproduce it in psql)
 2. Measure.                                  (EXPLAIN ANALYZE + \timing on)
 3. State a hypothesis.                       ("no index on customer_id")
 4. Change ONE thing.                         (CREATE INDEX ...)
 5. Re-measure.                               (same EXPLAIN ANALYZE)
 6. Keep if it helped, revert if it did not.  (git for schema / migrations)
```

**Why it matters:** changing two things at once makes the result unattributable. It is the same discipline as a controlled ML experiment: one variable, before/after, reproducible.

---

### 3. Query Anti-Patterns - The Six That Cause Most Pain

#### 3.1 Missing index on a `WHERE` / `JOIN` column -> sequential scan

The default. Without an index the engine reads every row. This is exactly the `Rows Removed by Filter: 99994` above. Fix: index the filtered/joined column (see 4).

#### 3.2 N+1 queries - one query per row in a loop

The single most common app-level DB bug. [-> `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)

```js
// N+1: 1 query for the list + N more inside the loop = 501 round-trips
const orders = await knex('orders').limit(500);
for (const o of orders) {
  o.customer = await knex('customers').where({ id: o.customer_id }).first();
}

// 1 query - join and select what you need
const orders = await knex('orders')
  .join('customers', 'customers.id', 'orders.customer_id')
  .select('orders.id', 'orders.total', 'customers.name as customer_name')
  .limit(500);
```

Each query costs a network round-trip plus a plan. Five hundred of them is five hundred times that, and it is invisible in a demo with ten rows. Count your queries per request - that number is the fingerprint.

#### 3.3 `SELECT *` over a fat row - transfer and planner waste

```sql
-- pulls a 5 KB JSON blob you never render
SELECT * FROM articles WHERE id = 1;

-- fetch only the columns the view uses
SELECT id, title, LEFT(body, 200) AS excerpt FROM articles WHERE id = 1;
```

`SELECT *` forces the engine to read and de-TOAST wide columns and ships bytes the template discards. On a list of 500 rows this is megabytes of waste and a slower, fatter wire.

#### 3.4 Deep `OFFSET` pagination - the DB walks and discards

```sql
-- page 1000: the engine reads 20 000 rows and throws away 19 980
SELECT id, title FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 20000;
```

`OFFSET` is not a seek: Postgres walks the first 20 000 matching rows to *count* them, then discards them. It gets slower linearly as you page deeper. The fix is **keyset (cursor) pagination** - remember the last row's sort key and ask for the next ones:

```sql
-- seek straight to the cursor; constant cost regardless of depth
SELECT id, title, created_at FROM posts
WHERE (created_at, id) < ($1, $2)   -- last seen (created_at, id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

This is the DB half of the pagination feature in the course app. [-> `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)

#### 3.5 Functions wrapping an indexed column - index silently ignored

```sql
-- index on email is NOT used: lower() changes the compared value
SELECT * FROM users WHERE lower(email) = 'ana@x.com';
```

The B-tree is sorted by `email`, not by `lower(email)`. The only ways out: don't wrap the column (normalise on write), or build a **matching functional index** `CREATE INDEX ... ON users (lower(email));`.

#### 3.6 Unbounded scans on huge tables

`SELECT count(*) FROM events;` or a report with no `LIMIT` on a 50-million-row table will run for seconds and pin a worker. Add a time bound (`WHERE created_at > now() - interval '30 days'`), precompute with a materialized view, or move it to an async job with a queue. [-> `../../software_architecture/01-event-driven-architecture-task-orchestration.md`](../../software_architecture/01-event-driven-architecture-task-orchestration.md)

---

### 4. Index Design Fixes - The Right Index, Not Just An Index

#### 4.1 Composite index and the leftmost-prefix rule

For a two-column filter, column **order matters**:

```sql
CREATE INDEX idx_orders_cust_created ON orders (customer_id, created_at DESC);

-- uses the index (leftmost column present)
SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC;

-- uses the index (both columns, range on the right one)
SELECT * FROM orders WHERE customer_id = 42 AND created_at > '2026-01-01';

-- cannot use it: created_at is not a leftmost prefix
SELECT * FROM orders WHERE created_at > '2026-01-01';
```

Rule: **equality columns first, then the range/sort column.** The index is sorted by `(customer_id, created_at)`, so a lookup must "enter" the tree through `customer_id` first. A separate `(created_at)` index would be needed for the last query - which is exactly why you do not index every column eagerly.

#### 4.2 Covering index - answer without touching the table

If the index already contains every column the query needs, the engine skips the heap fetch entirely (`Index Only Scan`):

```sql
-- columns not in the key are appended and carried along
CREATE INDEX idx_orders_covering ON orders (customer_id) INCLUDE (total, status);
```

#### 4.3 Partial index - index only the rows you query

```sql
-- only the small set of pending orders is indexed; a tiny, hot structure
CREATE INDEX idx_orders_pending ON orders (created_at)
  WHERE status = 'pending';
```

The index is smaller, faster, and cheaper to maintain because most rows are excluded. Perfect for "the queue of unfinished work" queries.

#### 4.4 When an index HURTS

An index is not free. Every `INSERT`/`UPDATE`/`DELETE` must update each index, so **write amplification** grows; each index costs disk; and a low-selectivity index (e.g. a `boolean`) that the planner judges unhelpful may be ignored anyway. The planner can also choose the *wrong* index when its row estimates are stale. So: index what you filter and join on, measure with `EXPLAIN ANALYZE`, and delete indexes nobody uses (`pg_stat_user_indexes.idx_scan = 0`).

#### 4.5 The index  DSA link

A B-tree index is a data structure with a cost model: $\Theta(\log n)$ lookup versus a $\Theta(n)$ scan, and a range scan that rides the sorted leaves. Choosing and ordering indexes is data-structure selection applied to storage. [-> `../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md)

---

### 5. Caching Layers - Move the Work Closer to the User

When indexing is not enough, cache. The layers are ordered by distance from the user; each is faster and *less specific* than the one below.

| Layer | Scope / stores | Freshness control | Invalidation | Per-user safe? |
|-------|----------------|-------------------|--------------|----------------|
| Browser | per-URL response/assets | `Cache-Control`, `ETag` | versioned URL or accept staleness | yes (private) |
| Reverse proxy (nginx) | full rendered pages | `proxy_cache_valid` TTL | purge file or wait out TTL | only with session bypass |
| App cache (`node-cache`) | objects, one process | TTL in code | TTL, `del`, restart | per-process |
| Shared cache (Redis) | query results / fragments | TTL + explicit keys | `DEL` on write / versioned keys | key by user id |
| Database | indexes, materialized views | planner stats | auto on write; matview `REFRESH` | yes |

#### 5.1 Cache-aside (lazy population) with pseudocode

The default pattern: read cache -> on miss read DB -> write result back with a TTL.

```
function get(key, loader, ttl):
    value = cache.get(key)
    if value is not None:
        return value              # fast path: no DB
    value = loader()              # slow path: one DB query
    cache.set(key, value, ttl)    # populate lazily
    return value

function updateProduct(p):        # write path
    db.update(p)
    cache.del("product:" + p.id)  # invalidate so the next read repopulates
```

```js
// Redis, runnable: npm i ioredis
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL || 'redis://127.0.0.1:6379');

export async function cached(key, ttlSeconds, loader) {
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);
  const value = await loader();
  await redis.set(key, JSON.stringify(value), 'EX', ttlSeconds);
  return value;
}
```

#### 5.2 The invalidation problem - "the two hard things"

A cache is a bet that the answer will not change before the TTL expires. When the bet is wrong, users see stale data (a stale price) or, worse, another user's data (a leaked per-user page). Two safe strategies:

1. **TTL as a safety net, explicit `DEL` as the guarantee.** Write-through invalidation (`DEL` the key on every update) keeps correctness; the TTL bounds the damage if a `DEL` is ever missed.
2. **Version-prefixed keys.** Store `products:v3:list` and bump the version on deploy; the old entries simply age out and are never read again.

If you cannot answer "what makes this entry wrong, and who deletes it?", do not cache it yet.

#### 5.3 Choosing a layer

- **Query results -> Redis** when the same expensive read repeats across users.
- **Rendered fragments/pages -> nginx micro-cache** when anonymous traffic dominates.
- **Static assets -> browser/CDN** with content-hashed filenames and a long `max-age`.
- **The deepest cache is free:** the index you already have.

> **Why it matters:** Caching is an *architecture* decision, not a code tweak, because *where* you place the cache decides the floor of latency you can reach. A request answered from the browser never reaches your DB at all.

---

### 6. Connection Pooling & Concurrency

Every Postgres connection is a forked backend process with its own memory - expensive to create. The app therefore keeps a **pool** of reusable connections (Knex/`pg` does this automatically). [-> `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)

```
  app process                         postgres
     borrow/release    
    pool (N)        N backends       
                                 + max_connections
                       
```

#### 6.1 Sizing

Two limits interact:

- **Postgres `max_connections`** (default 100) is a global ceiling across *all* app instances.
- **Your pool `max`** (node-postgres default 10) is per app process.

A common starting formula is:

$$\text{pool size} \approx (\text{CPU cores} \times 2) + \text{effective disks}$$

The intuition: each active query occupies a core for CPU work and waits on a disk. More connections than that mostly queue - they add context-switching and memory, not throughput. Start there and measure; It is a starting point, not a law.

#### 6.2 Pool exhaustion - symptom and fix

Symptoms: the app "hangs," requests time out, logs show `remaining connection slots are reserved` or `too many clients already`, or a Knex `Knex: Timeout acquiring a connection.` Two causes:

1. **A leak** - a connection borrowed and never released (usually a missing `await`, or a transaction never committed/rolled back). Fix the code; always `try/finally` around transactions.
2. **Too many workers x too large a pool.** 8 instances x pool 20 = 160 > `max_connections`. Fix: lower the pool per instance, or put **PgBouncer** in front to multiplex many clients onto few server connections - essential for serverless, where each invocation might open a connection.

```js
// explicit, measured pool sizing
const knex = require('knex')({
  client: 'pg',
  connection: process.env.DATABASE_URL,
  pool: { min: 2, max: 10, acquireTimeoutMillis: 5000 },
});
```

#### 6.3 Read replicas & denormalization - the scaling path (and its price)

When reads genuinely outgrow one server:

- **Read replicas** stream a copy of the data; send `SELECT`s there, writes to the primary. *Price:* replication lag means a read can be stale, and your code must choose a connection per query.
- **Denormalization** stores a precomputed value (a counter, a cached join) to avoid a join. *Price:* the value can drift from the source of truth, and you own updating it.

Honesty check: **these add operational complexity.** Reach for them only when measurement proves you need them - after indexes, after caching, after fixing N+1. **When to stop tuning SQL:** when the remaining bottleneck is a data-model mismatch (a report that joins ten tables), the honest fix is a read-optimized table or a materialized view, not a cleverer index.

---

### 7. The Method - A Decision Sequence

```
 
  1. MEASURE      EXPLAIN ANALYZE  slow-query log         
                  Server-Timing  p95 latency               
 
                             
 
  2. FIX SCHEMA / INDEX     right index, right column order, 
                  kill N+1, SELECT only needed columns,     
                  keyset pagination                         
 
                             
 
  3. ADD CACHE    Redis cache-aside -> nginx page cache ->    
                  browser/CDN for assets. Invalidate on     
                  write.                                    
 
                             
 
  4. SCALE READS  pool tuning / PgBouncer -> read replica ->  
                  denormalize / materialized view           
 
```

**Course-relevant practical checklist:**

- [ ] Reproduce the slow page's queries in `psql`; run `EXPLAIN ANALYZE` on each.
- [ ] Add indexes on every `WHERE`/`JOIN`/`ORDER BY` column that shows up in a `Seq Scan`.
- [ ] Count queries per request; collapse any loop-of-queries into a join or `whereIn`.
- [ ] Replace `SELECT *` with explicit columns; replace deep `OFFSET` with keyset pagination.
- [ ] Add a Redis or `node-cache` cache-aside for the hottest shared read; `DEL` on write.
- [ ] Set an explicit pool `max`; ensure every borrowed connection and transaction is released.
- [ ] Re-measure after each step; stop when the target latency or throughput is met.

> **Why it matters (exam and career):** The syllabus wants the *method*, not a library. And every AI product - a RAG retriever hitting a document store, a feature store feeding a model - sits on a database under the same fan-in pressure. The engineer who can read a plan, name the query, and pick a fix is the one who ships systems that survive their first thousand users.

---

### 8. Worked Example - Missing Index, Before and After

Reproduce with the Dockerized Postgres from `02-postgresql-core-concepts.md`, 100 000 rows.

```sql
CREATE TABLE orders (
  id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  total       NUMERIC(12,2) NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

INSERT INTO orders (customer_id, total)
SELECT (g % 5000) + 1, (random() * 500)::numeric(12,2)
FROM generate_series(1, 100000) g;
```

**Before - no index on `customer_id`:**

```sql
EXPLAIN ANALYZE SELECT id, total FROM orders WHERE customer_id = 42;
```

```
Seq Scan on orders  (cost=0.00..1834.00 rows=20 width=12)
                    (actual time=0.014..12.104 rows=20 loops=1)
  Filter: (customer_id = 42)
  Rows Removed by Filter: 99980
Planning Time: 0.092 ms
Execution Time: 12.180 ms
```

Read: a `Seq Scan`, 100 000 rows touched, 99 980 discarded, 12 ms - and it grows linearly with the table.

**The one change:**

```sql
CREATE INDEX idx_orders_customer ON orders (customer_id);
ANALYZE orders;
```

**After - same query:**

```sql
EXPLAIN ANALYZE SELECT id, total FROM orders WHERE customer_id = 42;
```

```
Index Scan using idx_orders_customer on orders
  (cost=0.42..8.44 rows=20 width=12) (actual time=0.028..0.041 rows=20 loops=1)
  Index Cond: (customer_id = 42)
Planning Time: 0.110 ms
Execution Time: 0.052 ms
```

Read: an `Index Scan`, a handful of leaf accesses, no `Rows Removed`, **~230x faster** - and the cost is now logarithmic, so it holds as the table grows. That single plan difference is the entire lesson of this note.

---

## Tech Cross-References

- **PostgreSQL core - indexes and `EXPLAIN` are the same machinery.** This note's anti-patterns are the engine note's index trade-off (faster reads, slower writes, more disk) under real load. [-> `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)
- **Knex - the app emits every query here and owns the pool.** N+1 in a loop, `select('col')` instead of `*`, and `pool: { max }` are all Knex-level decisions. [-> `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)
- **DSA - a B-tree is a data structure with a cost model.** Choosing and ordering an index is data-structure selection; `EXPLAIN ANALYZE` is empirical complexity testing. [-> `../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md)
- **Web server - pagination is where deep `OFFSET` bites.** The keyset-pagination fix lives in this course feature; the DB and the route must agree on the cursor. [-> `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)
- **Architecture - caching is a placement decision.** The five cache layers and their invalidation costs are covered from the whole-request perspective. [-> `../../software_architecture/04-web-performance-caching-and-bottlenecks.md`](../../software_architecture/04-web-performance-caching-and-bottlenecks.md)

---

## Research Journey

- **Why:** I could write a Knex query that worked on my ten test rows and still watch the page crawl once the seed script loaded 100 000. That gap - "correct query" versus "correct *and* fast query" - is what this note exists to close. The course's claim that performance is "mostly architecture plus the DB" sounded like a slogan until I had a page where 1.7 of its 1.8 seconds was one unindexed lookup.
- **Struggle:** Two points. First, `EXPLAIN` output looked like noise until I learned to read only four things: node type, estimated-vs-actual rows, rows removed, and actual time. Second, the counter-intuitive result that **adding an index can make the *system* slower** - faster reads but slower writes and more disk - forced me to stop thinking "index = good" and start thinking "index = a trade with a cost model."
- **Aha moment:** Watching `Seq Scan  Rows Removed by Filter: 99980` collapse to `Index Scan  Execution Time: 0.052 ms`. The DSA B-tree, which I had only drawn on paper, was suddenly visible in a plan and worth 230 times the wall-clock. The second aha was that N+1 - a loop that looks like ordinary JavaScript - is really 500 network round-trips, and joins/batching are how you delete them.
- **Career link:** Every serious AI product I want to build sits on a data system: a vector/document store for RAG, a feature store for training, an event log for analytics. They all suffer the same fan-in, the same N+1, the same cache-invalidation question. Reading a query plan and choosing between an index, a cache, and a replica is daily AI-engineering work, not a DBA's private skill.

## Related Topics

- [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) - indexes, `EXPLAIN`, MVCC, and the pooling pitfall this note expands
- [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) - the query builder and pool that emit everything here
- [`07-postgresql-full-text-search.md`](07-postgresql-full-text-search.md) - text-search queries are subject to the same plan-and-index discipline
- [`../../software_architecture/04-web-performance-caching-and-bottlenecks.md`](../../software_architecture/04-web-performance-caching-and-bottlenecks.md) - the end-to-end view: where the DB millisecond sits in the request waterfall
- [`../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md) - the pagination feature that must adopt keyset pagination
- [`../../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md) - an expensive unbounded query is also a denial-of-service vector; rate-limiting and query bounds are a security control
- [SUGGESTED] `09-connection-pooling-and-pgbouncer.md` - *why it's important:* pool sizing and PgBouncer deserve their own operational note, and they are mandatory for serverless Postgres. *Folder:* `dev_foundation/databases/`.
- [SUGGESTED] `10-observability-and-slow-query-monitoring.md` - *why it's important:* this note measures by hand; in production you need `pg_stat_statements`, the slow-query log, and dashboards that surface the top queries before users do. *Folder:* `dev_foundation/databases/` or `software_architecture/`.

## Open Questions

- [ ] For the course project, what is the p95 of each route's DB segment - and which single query dominates it?
- [ ] At what scale is a Redis cache-aside worth the invalidation risk versus letting Postgres's own buffer cache and a covering index do the work?
- [ ] What is the exact point where a keyset cursor becomes awkward (multi-column sorts, mutable sort keys), and what is the honest fallback?
- [ ] How should an AI feature store (embeddings table, vector index) be cached and pooled differently from an OLTP table?
- [ ] Does `pg_stat_statements` change my method - should "read the top-10 queries by total time" replace my hand-run `EXPLAIN` as the first step?

---
*The database is not slow. One query is. Find it, measure it, and change one thing.*

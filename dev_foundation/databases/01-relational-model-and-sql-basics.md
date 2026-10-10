# Relational Model & SQL Basics — Thinking in Sets

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#SQL`, `#PostgreSQL`, `#RelationalModel`, `#Joins`, `#Normalization`, `#DevTools`
**References:** PostgreSQL 16 documentation (§4 SQL Syntax, §2 The SQL Language), MDN "SQL" glossary, ISO/IEC 9075 (SQL standard), Codd (1970) *A Relational Model of Data for Large Shared Data Banks*
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`../../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md) (run `psql` from a shell), [`../../git_github/01-version-control-concepts.md`](../../git_github/01-version-control-concepts.md) (mental model of versioned state)

---

## 📌 Key Takeaways (TL;DR)

- **A relational database is a set of tables, and SQL is a set-based language.** You *declare* the result you want; the engine's planner decides how to compute it (seq scan, index scan, hash join). You almost never write "loop over rows" — you write "give me the set of rows where …".
- **Before typing any command, answer one question: "what set am I producing?"** Most SQL mistakes (missing `ON`, missing `GROUP BY`, `WHERE x = NULL`) are really mistakes about which set you asked for.
- **A SQL table is a *bag*, not a true set.** Duplicates are allowed and row order is not guaranteed. If you need a set, say `SELECT DISTINCT`; if you need order, say `ORDER BY`.
- **Three families of statements:** DDL defines structure (`CREATE/ALTER/DROP`), DML changes rows (`INSERT/UPDATE/DELETE`), DQL reads rows (`SELECT`).
- **`WHERE x = NULL` is never true.** `NULL` means "unknown", so comparisons produce a third truth value (`UNKNOWN`), and the row is silently filtered out. Use `IS NULL`.
- **`UPDATE`/`DELETE` without `WHERE` is nuclear** — it affects every row. Always `SELECT` the same predicate first.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 A database is a set of tables

A relational database is a **collection of relations**. Informally, a relation is a **table**: a named set of **rows** (tuples) described by named **columns** (attributes), where every column has a **type**.

```
Database "shop"
├── table customers   (id, email, name, created_at)
├── table orders      (id, customer_id, total, placed_at)
└── table order_items (id, order_id, product_id, qty, unit_price)
```

The word "relational" does **not** mean "related tables". It comes from the mathematical notion of a *relation* — a set of tuples. The foreign keys between tables are a *consequence* of the model, not its definition.

#### 1.2 SQL is declarative and set-based

In an imperative language you say *how*: `for row in rows: if row.x > 5: result.append(row)`. In SQL you say *what*:

```sql
SELECT name FROM customers WHERE signup_year > 2020;
```

You did not say which index to use, in what order to read the file, or whether to parallelise. The **query planner** makes those choices. This is the single most important shift in thinking:

> **You describe a set. The engine produces it.**

This is why SQL queries feel like they "read backwards": `SELECT` (the columns of the result) comes first, but it is logically evaluated **last** — after `FROM`, `WHERE`, `JOIN`, and `GROUP BY` have produced the candidate rows.

#### 1.3 The logical evaluation order (why it matters)

The clauses are written in one order and evaluated in another:

```
FROM  →  WHERE  →  GROUP BY  →  HAVING  →  SELECT  →  ORDER BY  →  LIMIT
```

Two rules fall out of this order, and they explain 80% of beginner errors:

1. **`WHERE` runs before `SELECT`**, so you cannot filter on a column alias you only created in `SELECT`.
2. **`WHERE` runs before `GROUP BY`**, so `WHERE` filters *rows*, while `HAVING` filters *groups* (see §5).

#### 1.4 Why this model matters

Because the model is set-based, one query can run against 100 rows or 100 million rows with the same SQL text — only the plan changes. That scalability, plus the guarantee that a transaction either fully happens or does not (§02), is why relational databases have outlived every "X is dead" prediction since 1970.

---

### 2. Tables, Keys & Constraints

A table is not just a grid; it carries **rules** that the engine enforces on every write.

#### 2.1 Columns and the common data types

| Type family | Examples | Use for | Watch out for |
|-------------|----------|---------|---------------|
| Integer | `INTEGER`, `BIGINT`, `SMALLINT` | Counts, ids, quantities | Overflow on `SMALLINT` |
| Text | `TEXT`, `VARCHAR(n)`, `CHAR(n)` | Names, emails, slugs | `CHAR` pads with spaces |
| Exact decimal | `NUMERIC(p,s)` | Money, prices | Slower than float (exact = more work) |
| Floating point | `REAL`, `DOUBLE PRECISION` | Scientific measurement | **Never use for money** (rounding) |
| Boolean | `BOOLEAN` | `true`/`false`/`NULL` | No implicit `0`/`1` (that is MySQL) |
| Date/time | `DATE`, `TIMESTAMP`, `TIMESTAMPTZ` | Events, deadlines | The timezone trap — see [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) §3 |
| Structured | `JSONB` | Flexible/nested data | Can't index arbitrary keys by default |
| Array | `INTEGER[]`, `TEXT[]` | Tags, small ordered lists | Not a substitute for a join table |

#### 2.2 Keys

```sql
-- PRIMARY KEY = uniqueness + NOT NULL + identity for the row
CREATE TABLE customers (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email       TEXT NOT NULL UNIQUE,
    name        TEXT NOT NULL,
    credit      NUMERIC(10,2) NOT NULL DEFAULT 0 CHECK (credit >= 0),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

- **`PRIMARY KEY`** — uniquely identifies each row, implies `NOT NULL`, and is how other tables point at this one.
- **`FOREIGN KEY`** — points at a primary key (or unique key) in another table and enforces **referential integrity**: you cannot insert an order for a nonexistent customer, and (depending on the action) you cannot delete a customer who has orders.

```sql
CREATE TABLE orders (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id),
    total       NUMERIC(10,2) NOT NULL CHECK (total >= 0),
    placed_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### 2.3 The other constraints

| Constraint | Guarantees | Example |
|------------|-----------|---------|
| `NOT NULL` | Column always has a value | `name TEXT NOT NULL` |
| `UNIQUE` | No two rows share the value (an implicit index is created) | `email TEXT UNIQUE` |
| `CHECK` | A boolean rule must hold | `CHECK (credit >= 0)` |
| `DEFAULT` | Value used when none is supplied | `DEFAULT now()` |
| `PRIMARY KEY` | `UNIQUE` + `NOT NULL` for the whole row | `id … PRIMARY KEY` |
| `FOREIGN KEY` | Referential integrity to another table | `REFERENCES customers(id)` |

**Why this matters:** constraints are the database keeping *you* honest. If a rule lives only in application code, a second writer (a script, an admin, a migration) can violate it. A `CHECK (credit >= 0)` at the storage layer cannot be bypassed without a schema change.

> **Note:** `REFERENCES` (FK) does **not** automatically create an index on the referencing column in PostgreSQL. Add one yourself if you join/filter on it — see [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) §5.

---

### 3. The SQL Families

| Family | Statements | Effect |
|--------|-----------|--------|
| **DDL** — Data Definition | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Change the *structure* (schema) |
| **DML** — Data Manipulation | `INSERT`, `UPDATE`, `DELETE` | Change the *rows* |
| **DQL** — Data Query | `SELECT` | Read rows (never modifies) |

#### 3.1 DDL — define structure

```sql
CREATE TABLE categories (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  TEXT NOT NULL UNIQUE
);

ALTER TABLE orders ADD COLUMN note TEXT;      -- add a column
ALTER TABLE orders ALTER COLUMN total SET NOT NULL;
DROP TABLE categories;                          -- destroy table + data
```

#### 3.2 DML — change rows

```sql
INSERT INTO customers (email, name) VALUES
  ('ana@example.com', 'Ana'),
  ('bao@example.com', 'Bao');

UPDATE customers SET credit = credit + 50 WHERE email = 'ana@example.com';

DELETE FROM customers WHERE email = 'bao@example.com';
```

#### 3.3 DQL — `WHERE`, operators, `ORDER BY`, `LIMIT`

```sql
SELECT name, credit
FROM   customers
WHERE  credit >= 100 AND name LIKE 'A%'
ORDER  BY credit DESC, name ASC
LIMIT  10 OFFSET 20;
```

- **Operators:** `=`, `<>`/`!=`, `<`, `<=`, `>`, `>=`, `BETWEEN a AND b`, `IN (…)`, `LIKE`/`ILIKE` (`%`=any run, `_`=one char), `IS NULL`, `EXISTS`.
- **`ORDER BY`** is required for deterministic output; without it the row order is unspecified. Add a tiebreaker (`name`) so the order is total.
- **`LIMIT n OFFSET m`** — "skip `m`, take `n`". It reads *and discards* the first `m` rows, so deep `OFFSET` is O(m) work even with an index; keyset pagination (`WHERE id > :last ORDER BY id LIMIT n`) is the fix.

---

### 4. JOINs

A join combines rows from two tables by matching a condition. Think of it as producing a new set.

```sql
-- INNER JOIN: only customers WITH matching orders
SELECT c.name, o.total
FROM   customers c
JOIN   orders o ON o.customer_id = c.id;

-- LEFT JOIN: every customer, orders where they exist, NULL otherwise
SELECT c.name, o.total
FROM   customers c
LEFT JOIN orders o ON o.customer_id = c.id;
```

| Join | Keeps |
|------|-------|
| `INNER JOIN` | Only rows with a match on **both** sides |
| `LEFT JOIN` | All left rows; right columns are `NULL` when no match |
| `RIGHT JOIN` | All right rows; left columns `NULL` when no match |
| `FULL JOIN` | All rows from both sides; missing side is `NULL` |

**`ON` vs `USING`:** `ON o.customer_id = c.id` works for any columns. `USING (id)` is shorthand when both tables have a column of the same name; it keeps a single copy of that column in the output.

#### 4.1 Self-join — an employee reporting tree

A self-join joins a table to itself. A management hierarchy is the canonical case:

```sql
CREATE TABLE employees (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    manager_id INTEGER REFERENCES employees(id)   -- points to another employee
);

-- each employee with their manager's name
SELECT e.name AS employee, m.name AS manager
FROM   employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

| employee | manager |
|----------|---------|
| Ana | NULL |
| Bao | Ana |
| Chau | Ana |

#### 4.2 The cartesian-product danger

**A join with no `ON` (or a wrong `ON`) produces every pairing** — an $m \times n$ cartesian product:

```sql
SELECT c.name, o.total
FROM   customers c, orders o;   -- ⚠️ NO join condition → every customer × every order
```

3 customers × 4 orders = 12 rows, none of them meaningful. On real tables this explodes to millions of rows, exhausts memory, and is one of the classic ways a "small change" takes down a report. `FULL JOIN` has the same trap when its `ON` is wrong.

---

### 5. Aggregation

Aggregates collapse a set of rows into one value.

```sql
SELECT
    COUNT(*)              AS n_orders,
    COUNT(DISTINCT c.id)  AS n_customers,
    SUM(o.total)          AS revenue,
    AVG(o.total)          AS avg_order,
    MIN(o.total)          AS smallest,
    MAX(o.total)          AS largest
FROM orders o JOIN customers c ON c.id = o.customer_id;
```

#### 5.1 `GROUP BY` — one result row per group

```sql
SELECT customer_id, COUNT(*) AS n_orders, SUM(total) AS revenue
FROM   orders
GROUP  BY customer_id
ORDER  BY revenue DESC;
```

**The rule:** every column in `SELECT` that is *not* inside an aggregate **must appear in `GROUP BY`** (or be functionally dependent on the grouped PK). This is not pedantry — if `name` were neither grouped nor aggregated, which of the many names in the group should be returned? PostgreSQL raises `column "…" must appear in the GROUP BY clause` for exactly this.

#### 5.2 `HAVING` vs `WHERE`

| Clause | Filters | Runs | Can use aggregates? |
|--------|---------|------|---------------------|
| `WHERE` | individual **rows** | before grouping | **No** |
| `HAVING` | **groups** | after grouping | **Yes** |

```sql
SELECT customer_id, SUM(total) AS revenue
FROM   orders
WHERE  total > 10            -- drop tiny orders BEFORE grouping
GROUP  BY customer_id
HAVING SUM(total) > 500      -- keep only big *customers* AFTER grouping
ORDER  BY revenue DESC;
```

Putting the `total > 10` test in `HAVING` would still work but would aggregate more rows than necessary — `WHERE` is cheaper because it removes rows earlier.

---

### 6. NULL Semantics — Three-Valued Logic

`NULL` is not zero and not an empty string. It means **unknown / not applicable**. Any comparison with `NULL` yields `UNKNOWN`, and `WHERE` keeps only rows where the predicate is `TRUE`.

| `AND` | TRUE | FALSE | UNKNOWN |
|-------|------|-------|---------|
| **TRUE** | TRUE | FALSE | UNKNOWN |
| **FALSE** | FALSE | FALSE | FALSE |
| **UNKNOWN** | UNKNOWN | FALSE | UNKNOWN |

Consequences:

```sql
SELECT * FROM customers WHERE credit = NULL;      -- ❌ 0 rows (always UNKNOWN)
SELECT * FROM customers WHERE credit IS NULL;     -- ✅ correct
SELECT * FROM customers WHERE credit IS NOT NULL;

-- NOT IN with a NULL in the list is a trap:
SELECT * FROM customers WHERE id NOT IN (SELECT customer_id FROM orders);
-- if ANY customer_id is NULL, this returns ZERO rows (UNKNOWN can never be TRUE)
```

Aggregates ignore `NULL` (`COUNT(col)` counts non-null values; `COUNT(*)` counts rows), and `SUM/AVG/MIN/MAX` skip `NULL` — so `AVG` is over the non-null values only, which may surprise you. In a `LEFT JOIN`, the unmatched side is filled with `NULL`, which is why `WHERE o.total > 100` after a `LEFT JOIN` silently converts it into an `INNER JOIN`.

---

### 7. Normalization (Introduction)

Normalization is the process of organising columns into tables so that each fact is stored **once**. The goal is to avoid **update anomalies**.

Take one fat table:

| order_id | customer | customer_email | product | qty |
|----------|----------|----------------|---------|-----|
| 1 | Ana | ana@x.com | Keyboard | 1 |
| 1 | Ana | ana@x.com | Mouse | 2 |
| 2 | Bao | bao@x.com | Keyboard | 1 |

Problems (anomalies):

- **Update anomaly:** Ana changes her email → must update every order row, and missing one leaves contradictory data.
- **Insertion anomaly:** cannot record a product nobody has ordered yet.
- **Deletion anomaly:** deleting Bao's only order forgets Bao entirely.

**Decompose:**

```
customers(id, name, email)          -- one row per person → email stored once
products(id, name)                  -- one row per product
orders(id, customer_id, placed_at)  -- one row per order
order_items(order_id, product_id, qty)   -- the many-to-many bridge, PK = (order_id, product_id)
```

The normal forms, informally:

| Form | Rule | The fat table violates it because… |
|------|------|-----------------------------------|
| **1NF** | Atomic values, no repeating groups | a "products: Keyboard, Mouse" cell is a list, not a value |
| **2NF** | 1NF + no partial dependency on part of a composite key | `customer` depends on `order_id`, not on the full `(order_id, product)` key |
| **3NF** | 2NF + no transitive dependency (non-key → non-key) | `customer_email` depends on `customer`, not directly on the key |

**Practical takeaway (not dogma):** split repeated data into its own table and point at it with a foreign key; store each fact once. Over-normalisation (splitting a single always-together pair) costs joins, so real schemas balance — but the default for a learner is: **if a value repeats across rows, it probably belongs in another table.**

---

### 8. The `psql` Client

`psql` is the interactive terminal for PostgreSQL. The backslash commands are **client meta-commands** (not SQL):

| Command | Effect |
|---------|--------|
| `\l` or `\l+` | List databases |
| `\c dbname` | Connect to another database |
| `\dt` | List tables in the current schema |
| `\d tablename` | Describe a table (columns, types, indexes, FKs) |
| `\d+ tablename` | Same, with storage/size details |
| `\dn` | List schemas |
| `\du` | List roles |
| `\x` | Toggle expanded (vertical) output — great for wide rows |
| `\timing` | Toggle per-statement timing |
| `\i file.sql` | Execute a SQL file |
| `\e` | Open the current query buffer in `$EDITOR` |
| `\q` | Quit |

Run a one-off without entering the shell:

```bash
psql -h localhost -U shopuser -d shop -c '\dt'
psql -h localhost -U shopuser -d shop -f schema.sql
```

---

### 9. Command / Code Cheat Sheet

| Command | What it does | When to use | Example | Expected output |
|---------|--------------|-------------|---------|-----------------|
| `CREATE TABLE …` | Define a table + constraints | Schema setup / migration | `CREATE TABLE t (id INT PRIMARY KEY, name TEXT NOT NULL);` | `CREATE TABLE` |
| `INSERT INTO … VALUES …` | Add rows | Seed / app write | `INSERT INTO t (id,name) VALUES (1,'Ana');` | `INSERT 0 1` |
| `SELECT … WHERE …` | Filter rows to a set | Any read | `SELECT name FROM t WHERE id = 1;` | ` Ana` |
| `SELECT … ORDER BY … LIMIT …` | Sort + paginate | Top-N, paging | `SELECT * FROM t ORDER BY id DESC LIMIT 1;` | newest row |
| `… JOIN … ON …` | Combine matching rows | Related data | `SELECT t.name,o.total FROM t JOIN o ON o.t_id=t.id;` | aligned rows |
| `… GROUP BY … HAVING …` | Aggregate per group, filter groups | Reports | `SELECT t_id,COUNT(*) FROM o GROUP BY t_id HAVING COUNT(*)>1;` | only busy ids |
| `UPDATE … SET … WHERE …` | Modify selected rows | Corrections/state change | `UPDATE t SET name='Ann' WHERE id=1;` | `UPDATE 1` |
| `DELETE FROM … WHERE …` | Remove selected rows | Cleanup | `DELETE FROM t WHERE id=1;` | `DELETE 1` |
| `SELECT DISTINCT col …` | De-duplicate a column | "Which values exist?" | `SELECT DISTINCT t_id FROM o;` | unique ids |
| `SELECT col … , col IS NULL AS missing` | Inspect nulls | Data-quality check | `SELECT id, credit IS NULL FROM t;` | boolean column |

---

### 10. Practical Lab — Build a Tiny Shop and Query It

Everything below is runnable and has a verifiable expected output. Work in a scratch database.

**Step 0 — create a database and load the schema.** Save as `lab.sql`:

```sql
DROP TABLE IF EXISTS order_items, orders, products, customers CASCADE;

CREATE TABLE customers (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE products (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  TEXT NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
);

CREATE TABLE orders (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id),
    placed_at   DATE NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE order_items (
    order_id   INTEGER NOT NULL REFERENCES orders(id),
    product_id INTEGER NOT NULL REFERENCES products(id),
    qty        INTEGER NOT NULL CHECK (qty > 0),
    PRIMARY KEY (order_id, product_id)
);

INSERT INTO customers (name, email) VALUES
  ('Ana', 'ana@x.com'), ('Bao', 'bao@x.com'), ('Chau', 'chau@x.com');
INSERT INTO products (name, price) VALUES
  ('Keyboard', 49.00), ('Mouse', 19.00), ('Monitor', 199.00);
INSERT INTO orders (customer_id, placed_at) VALUES
  (1, '2026-01-02'), (1, '2026-02-05'), (2, '2026-02-07');
INSERT INTO order_items (order_id, product_id, qty) VALUES
  (1, 1, 1), (1, 2, 2), (2, 3, 1), (3, 1, 1);
```

```bash
createdb shop_lab
psql -d shop_lab -f lab.sql
```

**Step 1 — who bought what (INNER JOIN):**

```sql
SELECT c.name, p.name AS product, i.qty
FROM order_items i
JOIN orders     o ON o.id = i.order_id
JOIN customers  c ON c.id = o.customer_id
JOIN products   p ON p.id = i.product_id
ORDER BY c.name, p.name;
```

```
 name | product  | qty
------+----------+-----
 Ana  | Keyboard |   1
 Ana  | Monitor  |   1
 Ana  | Mouse    |   2
 Bao  | Keyboard |   1
(4 rows)
```

**Step 2 — every customer, even those with no orders (LEFT JOIN):**

```sql
SELECT c.name, COUNT(o.id) AS n_orders
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.name
ORDER BY c.name;
```

```
 name | n_orders
------+----------
 Ana  |        2
 Bao  |        1
 Chau |        0
(3 rows)
```

Chau appears with `0` — a `LEFT JOIN` plus `COUNT(o.id)` (not `COUNT(*)`) is the standard "count including zeros" idiom.

**Step 3 — revenue per customer, only big spenders (WHERE + GROUP BY + HAVING):**

```sql
SELECT c.name, SUM(p.price * i.qty) AS revenue
FROM order_items i
JOIN orders    o ON o.id = i.order_id
JOIN customers c ON c.id = o.customer_id
JOIN products  p ON p.id = i.product_id
GROUP BY c.name
HAVING SUM(p.price * i.qty) > 100
ORDER BY revenue DESC;
```

```
 name | revenue
------+---------
 Ana  |  286.00
(1 row)
```

**Step 4 — prove the NULL trap:**

```sql
SELECT COUNT(*) FILTER (WHERE email IS NULL) AS missing_email FROM customers;
```

```
 missing_email
---------------
             0
(1 row)
```

**Step 5 — the cartesian product (do this once, and never again):**

```sql
SELECT COUNT(*) AS n FROM customers, products;
```

```
 n
----
 9
(1 row)
```

3 customers × 3 products = 9 rows — none of them meaningful. This is the "missing `ON`" bug in miniature.

---

### 11. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `WHERE col = NULL` | Query returns 0 rows silently | `WHERE col IS NULL` |
| `NOT IN (subquery)` where the subquery can return `NULL` | Query returns 0 rows | Use `NOT EXISTS`, or add `WHERE x IS NOT NULL` |
| Missing `ON` in a join (`FROM a, b`) | Huge row count / OOM | Always join with `ON`; prefer explicit `JOIN` syntax |
| `SELECT` non-aggregated column not in `GROUP BY` | `column "…" must appear in the GROUP BY clause` | Add it to `GROUP BY`, or aggregate it |
| `UPDATE`/`DELETE` without `WHERE` | Every row changed/deleted | Run the same predicate as `SELECT` first; wrap in a transaction |
| Comparing `TEXT` to `INTEGER` | `operator does not exist: text = integer` | Cast explicitly: `WHERE code::int = 5`, or fix the schema type |
| Deep `OFFSET` pagination | Slow page ~10 000 | Keyset pagination: `WHERE id > :last ORDER BY id LIMIT n` |
| `LEFT JOIN` then `WHERE right.col = x` | Becomes an `INNER JOIN` (rows vanish) | Put the condition in `ON`, or accept it and use `INNER JOIN` |
| `COUNT(*)` when counting the nullable side of a `LEFT JOIN` | Count shows 1 for "no order" | `COUNT(o.id)` counts non-null only |
| `SELECT *` in production code | Fragile when columns change | List columns explicitly |
| Assuming row order without `ORDER BY` | Flaky tests / wrong "first" row | Always `ORDER BY`, with a tiebreaker |
| `AVG(col)` where some rows are `NULL` | Average looks too high | `NULL`s are skipped; use `COALESCE(col, 0)` if you mean zero |

---

### 12. Comparison Table — SQL vs an Imperative Loop

| Dimension | Set-based SQL | Row-at-a-time loop (e.g. Python) |
|-----------|---------------|----------------------------------|
| What you write | The desired result set | The steps to compute it |
| Where it runs | Inside the database engine | In the app, pulling rows over the wire |
| Optimiser | Planner rewrites/uses indexes automatically | You hand-optimise |
| Scalability | Same query on 1 000 or 1 000 000 000 rows | Usually degrades with N+1 round-trips |
| Debugging | `EXPLAIN ANALYZE` shows the plan | A debugger / print statements |
| Best for | Cold data, aggregates, joins over large sets | Complex per-row logic, external APIs |
| Risk | Surprise cartesian products | N+1 queries, memory blowups |

The rule of thumb: **filter, join, and aggregate in the database; do per-row orchestration in code.**

---

## 🧪 Practice Tasks

> No solutions provided. Do them, then ask for a graded review or a harder variant.

- [ ] **Task 1 — Build and query a movie database.** Create `movies(id, title, year, rating)` and `genres(id, name)`, with a `movie_genres(movie_id, genre_id)` bridge table. Insert at least 8 movies. Write queries for: (a) all movies after 2000 ordered by rating desc; (b) each genre with its movie count; (c) genres with more than 2 movies.
- [ ] **Task 2 — Fix a broken join.** Take a copy of the `shop_lab` tables and write `SELECT c.name, COUNT(*) FROM customers c LEFT JOIN orders o ON o.customer_id = c.id GROUP BY c.name;`. Explain (in a comment) why every customer shows at least `1`, then fix the count.
- [ ] **Task 3 — Prove three-valued logic.** In a scratch table, insert a row with a `NULL` score. Run `SELECT * FROM t WHERE score = NULL`, `WHERE score IS NULL`, and `WHERE NOT (score = 5)`. Record how many rows each returns and explain the difference in two sentences.
- [ ] **Task 4 — Normalize a fat table.** Below is one denormalized table. Decompose it into 3NF tables, write the `CREATE TABLE` statements with keys and constraints, and state which anomaly each split removes.

    | invoice_id | client | client_city | item | item_price | qty |
    |------------|--------|-------------|------|-----------|-----|
    | 1 | Ana | Hanoi | Pen | 2.00 | 10 |
    | 1 | Ana | Hanoi | Book | 15.00 | 1 |
    | 2 | Bao | Da Nang | Pen | 2.00 | 3 |

- [ ] **Task 5 (harder) — Anti-join three ways.** Find customers who have **no** orders using (a) `NOT IN`, (b) `NOT EXISTS`, (c) `LEFT JOIN … WHERE o.id IS NULL`. Make one customer's `customer_id` `NULL` in a variant and show how `NOT IN` breaks while the other two still work.

---

## 🔗 Tech Cross-References

- **DSA — indexes are the data structure behind `WHERE`.** A `PRIMARY KEY` / `UNIQUE` constraint builds a **B-tree**, and a B-tree lookup is $\Theta(\log n)$ versus a full scan's $\Theta(n)$. That is the same cost model as the searching chapter in DSA, now with a disk underneath it. [→ `../dsa/05-data-structure-selection-guide.md`](../dsa/05-data-structure-selection-guide.md)
- **Git — transactions are "commits" for data.** `BEGIN` … `COMMIT` groups statements into one atomic change, and `ROLLBACK` is a reset to the last commit — the same all-or-nothing snapshot idea Git gives files. [→ `../../git_github/01-version-control-concepts.md`](../../git_github/01-version-control-concepts.md)
- **PostgreSQL core — this note is the vocabulary; note 02 is the engine.** Types, transactions, indexes, and `psql` behaviour against a real server live in the sibling note. [→ `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)
- **Knex — the course's query builder writes this same SQL.** Every Knex call compiles down to the `SELECT`/`JOIN`/`GROUP BY` shapes here; understanding the SQL is what lets you read Knex's generated queries. [→ `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)

## 📝 Research Journey

- **Why:** I can write a `for` loop in my sleep, but the first time I opened a real schema I froze — I kept looking for "the loop" that joined customers to orders, and there wasn't one. I realised I had been treating SQL as a clumsy imperative language with weird syntax instead of a *different kind of language*. This note is me rebuilding that mental model from the ground up, before touching Knex.
- **Struggle:** Two things genuinely hurt. First, the difference between `WHERE` and `HAVING` felt arbitrary until I internalised the logical evaluation order (`FROM → WHERE → GROUP BY → HAVING → SELECT`). Second, `NULL` — I had assumed `NULL` was just "empty", so `= NULL` returning zero rows looked like a bug, not like three-valued logic working correctly.
- **Aha moment:** The evaluation-order diagram turned SQL from a pile of rules into one rule that predicts behaviour. Once I could say "`WHERE` runs before groups exist", the `WHERE`/`HAVING` split, the alias restriction, and the `LEFT JOIN`-plus-`WHERE` collapse all became the same fact. A close second: watching `FROM customers, products` return exactly $m \times n$ rows and finally *seeing* the cartesian product instead of reading about it.
- **Career link:** An AI Engineer in economics/business lives and dies by data joins — customer × transaction × time, product × region × channel. Feature engineering *is* set manipulation, and it happens mostly in SQL, not in Python. If I cannot think in sets, I will pull massive tables into memory and do it the slow way; if I can, I will write the one query that returns exactly the training set. This is the vocabulary that the Knex chapter, the PostgREST layer, and every analytics pipeline assumes.

## 🔗 Related Topics

- [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) — the engine that runs the SQL you just learned: roles, transactions, indexes, types
- [`README.md`](README.md) — folder roadmap (relational model → engine → PostgREST → schema design)
- [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) — the course's JS query builder; it emits exactly these statements
- [`08-db-bottlenecks-indexing-and-caching.md`](08-db-bottlenecks-indexing-and-caching.md) — what happens when these queries meet a large table
- [SUGGESTED] `bridge-sql-to-dsa.md` — a bridge note mapping each SQL operation to its DSA data structure and cost (B-tree ↔ index, hash ↔ `HashJoin`, sort-merge join ↔ merge sort); it would make the "set has a cost model" idea explicit and link `databases/` ↔ `dsa/`.
- [SUGGESTED] `09-window-functions.md` — `ROW_NUMBER`, `RANK`, and running totals. Every "top-N per group" business question (top product per region) is painful with `GROUP BY` and trivial with window functions; it is the natural next step after aggregation.

## 🤔 Open Questions

- [ ] When is it worth **denormalizing** on purpose (storing a redundant `order_total` or a cached `customer_name`) for read speed, and what triggers the decision to reverse it?
- [ ] How do I *choose* between `NOT IN`, `NOT EXISTS`, and `LEFT JOIN … IS NULL` on performance rather than on correctness — does the planner always normalize them to the same plan?
- [ ] Is there a principled rule for how far to normalize a small app like Joblink before the join count hurts readability more than it helps?
- [ ] How do `JSONB` columns interact with normalization — when is a document-shaped column a legitimate 1NF value and when is it a hidden violation?
- [ ] What actually happens, mechanically, when `ORDER BY` is omitted and the engine returns rows "in whatever order"? Is it heap order, index order, or something else — and can it ever be relied on accidentally?

---
*You are not writing loops. You are describing a set — and the database plans the work.*

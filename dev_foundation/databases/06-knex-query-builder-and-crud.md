# Knex — Query Builder, Migrations & CRUD

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Knex`, `#SQL`, `#PostgreSQL`, `#CRUD`, `#Migrations`, `#NodeJS`
**References:** Knex.js documentation — "Query Builder", "Migrations", "Transactions", "Schemas"; `knexjs.org`; PostgreSQL docs on `RETURNING` and prepared statements (postgresql.org); the course syllabus §4.1–4.2 (PostgreSQL via Knex; CRUD)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`01-relational-model-and-sql-basics.md`](01-relational-model-and-sql-basics.md) (tables, keys, `SELECT`/joins), [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) (the engine Knex talks to); [`../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md) for the integration context

---

## 📌 Key Takeaways (TL;DR)

- **Knex is a query *builder*, not an ORM.** You chain JavaScript methods that each compile to a piece of SQL; Knex runs the assembled statement and hands you plain row objects. There are no "models" hiding the database from you — what you write is what runs.
- **The chain *expresses intent*, and every value is bound as a parameter.** `.where({ id })` compiles to `where "id" = $1` with `id` sent separately, so a value can never be re-parsed as SQL. This is the primary SQL-injection defence the course uses, straight from the wire protocol. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)
- **Migrations are version control for your schema.** Like Git commits for code, each migration is an ordered, reviewable, reversible change to the table structure — so "the schema" is never a mystery that only one machine's database reflects. [→ `../../git_github/README.md`](../../git_github/README.md)
- **Seeds are *data*, migrations are *structure*.** Seed files load fake/dev rows; migrations change tables. Mixing the two is how "my local DB works but prod doesn't".
- **On PostgreSQL you must ask for the row back with `.returning()`.** `insert` without it gives you only an affected-row count; `insert ... .returning('*')` gives the actual row (id, timestamps, defaults) in one round-trip.
- **The scope trap:** `.update()` and `.del()` with no `.where()` hit **every row**. The dangerous part is that they *succeed silently*. Always build the `where` first.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 A query builder is a SQL *constructor*, not a replacement

An **ORM** (Sequelize, TypeORM, Prisma) maps tables to classes and hides SQL. Knex does the opposite: it keeps SQL visible and *constructs* it from method calls.

```js
const rows = await knex('posts').where({ published: true }).orderBy('created_at', 'desc');
```

The chain above compiles to:

```sql
select * from "posts" where "published" = $1 order by "created_at" desc
```

and executes with `['true']` bound to `$1`. This is the whole mental model: **a JS object graph → a SQL string + a list of bound values → a result set.**

Three consequences follow, and they are the reasons the course uses it:

1. **Intent is expressed in the type system.** A typo in `.wherer(...)` is a JavaScript error the moment you run the file — not a runtime SQL failure buried in a log. You get a syntax check for free.
2. **Values are parameterised by construction.** You never concatenate user input into a string. Every `.where({ id: req.params.id })` becomes `$1`, so `1 OR 1=1` stays a (harmless) string value rather than becoming SQL. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)
3. **The result is data, not objects with magic.** A query resolves to an array of plain `{ column: value }` objects. There is no lazy-loading, no change-tracking, no `.save()`. You insert explicitly, you update explicitly. What you see is what happened.

> **The boundary to remember:** Knex does not manage your schema *shape* at runtime and does not cache rows. It is a thin, honest layer over SQL. When you need to know exactly which SQL ran, append `.toString()` to a builder and it prints the statement.

#### 1.2 Migrations are Git for the schema

A database has two lives: the **structure** (tables, columns, constraints, indexes) and the **data** (rows). Structure changes must happen in the same order on every machine — your laptop, your teammate's laptop, the CI server, production. Hand-running `CREATE TABLE` in `psql` breaks this the moment there are two developers.

A **migration** is a small file with an `up` (apply) and a `down` (undo) function. Knex records which migrations have run in a bookkeeping table, `knex_migrations`. Running `migrate:latest` applies only the ones not yet applied, in filename/timestamp order.

Git analogy:

| Git | Knex migrations |
|-----|-----------------|
| A commit describes a change | A migration describes a schema change |
| `git log` is the ordered history | The `knex_migrations` table is the ordered history |
| `git revert` undoes a commit | `migrate:rollback` runs the migration's `down` |
| Everyone clones the same history | Everyone runs `migrate:latest` to reach the same schema |
| You never edit pushed history | You never edit an already-applied migration — you add a new one |

**Why it matters:** the course's grading runs your repo on a clean machine. If the schema only exists in your local Postgres, there is no app. Migrations make the schema reproducible from the repo alone.

#### 1.3 Connection pooling is built in — you do not create connections

`knex({ client: 'pg', connection: ... })` lazily opens a **pool** of connections and reuses them. Each query borrows a connection, runs, and returns it. This is why Knex feels like a single object even though PostgreSQL sessions are expensive. Pool sizing and exhaustion are a real failure mode — see the pool config in §3 and the pitfalls table in §12. [→ `08-db-bottlenecks-indexing-and-caching.md`](08-db-bottlenecks-indexing-and-caching.md)

---

### 2. Mechanism (How It Works)

#### 2.1 From a method chain to rows — the pipeline

A Knex query is **lazy**: building the chain does nothing until you `await` it (or call `.then()` / `.toString()`). At that moment it runs this pipeline:

```
  knex('posts').where({id}).select('title')            ← build (in memory)
        │
        ▼
  compile  →  SQL string + bound values                 ← ".toString()" reveals this step
        │     select "title" from "posts" where "id" = $1
        │     bindings: [ 4 ]
        ▼
  pg driver  →  borrow a connection from the pool        ← pool checkout
        │
        ▼
  PostgreSQL  →  parse → plan (index vs seq scan) → execute
        │
        ▼
  rows  →  [ { title: '…' } ]  →  connection returned to pool
        │
        ▼
  Promise resolves with an array of plain objects
```

1. **Build.** Each chained method appends a clause to an internal builder object. It is a description of a query, not a query.
2. **Compile.** Rendering turns that object graph into a **SQL text** and a **separate list of bound values**. `.where({ id: 4 })` becomes `where "id" = $1` with `[4]`. The two never travel as one string — that separation is the injection defence (§1.1).
3. **Checkout.** The `pg` driver takes a connection from the pool and sends the parameterised statement.
4. **Execute.** PostgreSQL parses, chooses a plan, runs it, and returns rows.
5. **Resolve.** Knex wraps the rows in plain objects and resolves the promise.

**Why it matters:** because compile happens per execution, you can inspect any builder with `.toString()`, and you can reuse a builder (`const base = knex('posts').where({ published: true })`) and extend it — each extension recompiles. It also explains why a Knex query is a *thenable*, not a result: nothing touches the database until you consume it.

#### 2.2 How migrations are tracked

`migrate:latest` does not guess. It reads the `knex_migrations` table:

```
knex_migrations
┌────┬──────────────────────────────────┬───────┐
│ id │ name                             │ batch │
├────┼──────────────────────────────────┼───────┤
│ 1  │ 20261010120000_create_users.js   │ 1     │
│ 2  │ 20261010120100_create_posts.js   │ 1     │
└────┴──────────────────────────────────┴───────┘
```

The steps are:

1. Connect, ensure the `knex_migrations` table exists.
2. List every file in the migrations directory, sorted by filename (timestamp).
3. Compute the **pending** set = all files minus the names recorded in the table.
4. Run each pending migration's `up()` in order, all recorded under one new `batch` number.
5. Insert each name into the table as it succeeds.

`migrate:rollback` finds the **highest batch number**, runs those migrations' `down()` in reverse order, and deletes their rows. This is why the `down()` must be the true inverse of `up()`, and why editing an applied migration is meaningless — the table already records it as done.

---

### 3. Setup & Connection

Install the builder and the driver together — Knex is driver-agnostic, so `pg` is the driver that actually speaks the PostgreSQL protocol:

```bash
npm i knex pg
npm i -D dotenv          # keep credentials out of source
```

Create a `knexfile.js` at the project root. Each key is an environment; CLI commands pick one with `--env` (default `development`):

```js
// knexfile.js
require('dotenv').config();

module.exports = {
  development: {
    client: 'pg',
    connection: process.env.DATABASE_URL,      // postgres://user:pass@host:5432/db
    migrations: { directory: './db/migrations' },
    seeds:      { directory: './db/seeds' },
    pool: { min: 2, max: 10 },                 // keep the pool small and bounded
  },
  test: {
    client: 'pg',
    connection: process.env.TEST_DATABASE_URL,
    migrations: { directory: './db/migrations' },
  },
};
```

```ini
# .env  (git-ignored — never commit this)
DATABASE_URL=postgres://shopuser:shoppass@localhost:5432/shop
```

The connection string is the same URI from the engine note — the exact reason that note told you to standardise on it. [→ `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)

Then a single shared instance the whole app imports:

```js
// db/knex.js
const knex = require('knex');
const config = require('../knexfile.js');
module.exports = knex(config[process.env.NODE_ENV || 'development']);
```

**Why it matters:** one shared instance means one pool for the whole process. Creating a new `knex(...)` per request leaks a pool per request and exhausts the server's `max_connections` — the classic "works in dev, dies in prod" bug.

Apply the schema with the CLI:

```bash
npx knex migrate:latest
```

---

### 4. Migrations — Version-Controlled Schema

#### 3.1 Creating one

```bash
npx knex migrate:make create_users
npx knex migrate:make create_posts
```

Knex writes `db/migrations/20261010120000_create_users.js` — the timestamp prefix is the ordering. You fill in `up`/`down`:

```js
// db/migrations/20261010120000_create_users.js
exports.up = async function (knex) {
  await knex.schema.createTable('users', (table) => {
    table.increments('id').primary();          // serial/identity, primary key
    table.string('name', 120).notNullable();
    table.string('email', 200).notNullable().unique();
    table.timestamps(true, true);              // created_at, updated_at (timestamptz, default now())
  });
};

exports.down = async function (knex) {
  await knex.schema.dropTable('users');
};
```

Now the second migration, which references the first — this ordering is exactly what the timestamp prefix guarantees:

```js
// db/migrations/20261010120100_create_posts.js
exports.up = async function (knex) {
  await knex.schema.createTable('posts', (table) => {
    table.increments('id').primary();
    table.string('title', 200).notNullable();
    table.text('body');
    table.boolean('published').notNullable().defaultTo(false);
    table.integer('author_id')
         .unsigned()
         .notNullable()
         .references('id').inTable('users')
         .onDelete('CASCADE');                 // delete a user → delete their posts
    table.timestamps(true, true);
    table.index('author_id');                  // FK is NOT auto-indexed by PostgreSQL
  });
};

exports.down = async function (knex) {
  await knex.schema.dropTable('posts');        // reverse order: posts before users
};
```

#### 3.2 The schema builder vocabulary

| Builder call | SQL | Use for |
|--------------|-----|---------|
| `table.increments('id')` | `serial primary key` (identity) | auto-increment PK |
| `table.string('title', 200)` | `varchar(200)` | short bounded text |
| `table.text('body')` | `text` | unbounded text |
| `table.integer('n')` / `table.bigInteger('n')` | `integer` / `bigint` | counts, FKs |
| `table.decimal('price', 12, 2)` | `numeric(12,2)` | **money** — never `float` |
| `table.boolean('flag')` | `boolean` | flags |
| `table.timestamp('x', { useTz: true })` | `timestamptz` | instants |
| `table.timestamps(true, true)` | `created_at/updated_at timestamptz default now()` | audit columns |
| `table.unique(['a','b'])` | `unique (a,b)` | composite uniqueness |
| `...references('id').inTable('users')` | `foreign key` | referential integrity |

#### 3.3 The four commands you actually use

```bash
npx knex migrate:latest       # apply everything pending
npx knex migrate:rollback     # undo the last batch (runs down())
npx knex migrate:list         # what's applied vs pending
npx knex migrate:status       # human-readable status table
```

**Never edit a migration that has already been applied** to a shared database — the bookkeeping says it ran, so your edit will never execute. Add a new migration that alters the table instead. This is the same discipline as "don't rewrite pushed Git history". [→ `../../git_github/README.md`](../../git_github/README.md)

---

### 5. Seeds — Development Data

A **seed** is a script that loads rows. Generate and edit:

```bash
npx knex seed:make 01_users
npx knex seed:make 02_posts
```

```js
// db/seeds/01_users.js
exports.seed = async function (knex) {
  await knex.raw('TRUNCATE TABLE users RESTART IDENTITY CASCADE'); // CASCADE also clears posts
  await knex('users').insert([
    { name: 'Ana',  email: 'ana@example.com' },
    { name: 'Binh', email: 'binh@example.com' },
  ]);
};
```

```js
// db/seeds/02_posts.js
exports.seed = async function (knex) {
  await knex.raw('TRUNCATE TABLE posts RESTART IDENTITY CASCADE');
  await knex('posts').insert([
    { title: 'Hello world', body: 'First post', published: true,  author_id: 1 },
    { title: 'Draft notes', body: 'Work in progress', published: false, author_id: 1 },
    { title: 'SQL joins',    body: 'Inner vs outer',  published: true,  author_id: 2 },
  ]);
};
```

```bash
npx knex seed:run
```

**Seed vs migrate — the rule:**

| | Migration | Seed |
|---|-----------|------|
| Changes | **structure** (tables, columns, indexes) | **rows** (data) |
| Runs in production? | yes, always, forward-only | usually **no** — production has real data |
| Idempotent? | once per environment | written to be re-runnable (truncate-then-insert) |
| Example | `ALTER TABLE … ADD COLUMN` | `INSERT INTO users (…)` |

`TRUNCATE … RESTART IDENTITY CASCADE` is the idiomatic reset: it clears the tables and resets the id sequences so the seeded ids are stable and predictable, and `CASCADE` clears dependent tables too.

---

### 6. CRUD — The Four Operations

#### 5.1 Read

```js
// all columns, all rows
await knex('posts').select('*');

// choose columns
await knex('posts').select('id', 'title');

// filter — object form binds as parameters
await knex('posts').where({ published: true, author_id: 2 });

// operators + ordering + pagination
await knex('posts')
  .where('created_at', '>', '2026-01-01')
  .orderBy('created_at', 'desc')
  .limit(10)
  .offset(20);

// one row (object) instead of an array; `undefined` if not found
await knex('posts').where({ id: 1 }).first();

// one column as an array
await knex('posts').pluck('title');            // → ['Hello world', 'SQL joins', …]

// count
const [{ count }] = await knex('posts').count('id'); // count is a STRING on PG → see pitfalls
```

`.first()` and `.pluck()` are conveniences with a real simplification behind them: they collapse an array-of-rows into the single thing you usually want, so the call site shows intent.

#### 5.2 Create (and why `.returning()` matters)

```js
const [post] = await knex('posts')
  .insert({ title: 'New', body: 'Body', author_id: 1 })
  .returning('*');                             // → [{ id: 4, title: 'New', published: false, created_at: … }]
```

On PostgreSQL, **without** `.returning(...)` the insert resolves to an affected-row count, not the row. You would need a second query to learn the new `id`. `.returning('*')` (or `.returning(['id','created_at'])`) fetches exactly what the database generated, in the same round-trip. This is the builder exposing `RETURNING` from the engine note. [→ `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)

#### 5.3 Update

```js
const [updated] = await knex('posts')
  .where({ id: 4 })                            // ← THE SCOPE. Omit it and every row changes.
  .update({ published: true })
  .returning('*');
```

```js
// atomic relative update — the safe "decrement" pattern
await knex('posts').where({ id: 4 }).increment('view_count', 1);
```

#### 5.4 Delete

```js
const deleted = await knex('posts').where({ id: 4 }).del();  // → number of rows removed
```

```js
const removed = await knex('posts').where({ id: 4 }).del().returning('*'); // the row(s), for an audit log
```

#### 5.5 The scope trap, drawn

```
knex('posts').update({ published: true });        →  UPDATE posts SET published = true;          ❌ all rows
knex('posts').where({ id: 4 }).update(...);       →  UPDATE posts SET published = true WHERE id=$1; ✅
```

The dangerous property is that the first one **succeeds**. There is no error, no warning — you just published the entire site. Build the `where` *before* the mutation, every time.

---

### 7. Joins & Aggregates

Knex joins are the SQL joins from the basics note, in method form:

```js
// inner join: only posts whose author exists
await knex('posts')
  .join('users', 'posts.author_id', 'users.id')
  .select('posts.id', 'posts.title', 'users.name as author');

// left join: every user, even with zero posts (author is null)
await knex('users')
  .leftJoin('posts', 'users.id', 'posts.author_id')
  .select('users.name', 'posts.title');
```

```js
// count per author
await knex('posts')
  .select('author_id')
  .count('* as post_count')
  .groupBy('author_id');

// total views for a filtered set
await knex('posts').where({ published: true }).sum('view_count as total');
```

Note the alias (`as author`, `as post_count`): without it, both `id` columns collide and one silently wins in the result object — a bug that looks like "wrong data" rather than "wrong query".

**When to stay in the builder vs drop to raw:** use the builder for anything it models cleanly (joins, filters, group by, aggregates). Drop to `.whereRaw()`, `.select(knex.raw(...))`, or `knex.raw()` when you need a PostgreSQL feature Knex does not expose — full-text operators, window functions, `ON CONFLICT`. **Always pass bound values into raw**, never interpolate:

```js
// SAFE — value is bound as $1
await knex('posts').whereRaw('LOWER(title) LIKE ?', [`%${q.toLowerCase()}%`]);

// UNSAFE — user input concatenated into SQL
await knex('posts').whereRaw(`LOWER(title) LIKE '%${q}%'`);
```

---

### 8. Transactions

A **transaction** groups multiple statements so they all commit or all roll back — the engine's `BEGIN`/`COMMIT`/`ROLLBACK`, wrapped by Knex. The course's multi-step features (place order → decrement stock → record payment) need it.

The callback form is the one to prefer, because Knex commits on success and rolls back automatically if your function throws:

```js
await knex.transaction(async (trx) => {
  const [order] = await trx('orders')
    .insert({ customer_id: 1, total: 100.00 })
    .returning('*');

  // suppose this fails for some product — the throw rolls back the order too
  await trx('products').where({ id: 7 }).decrement('stock', 1);

  await trx('payments').insert({ order_id: order.id, amount: 100.00 });
});
```

The manual form makes the mechanics explicit:

```js
const trx = await knex.transaction();
try {
  await trx('orders').insert({ customer_id: 1, total: 100.00 });
  await trx('products').where({ id: 7 }).decrement('stock', 1);
  await trx.commit();
} catch (err) {
  await trx.rollback();                        // undo BOTH statements
  throw err;
}
```

**The `.transacting(trx)` rule:** inside a transaction, *every* statement must run on `trx`, not on the global `knex`. When a model function takes a query builder, pass `trx` through it:

```js
async function createOrder(trx, data) {
  return trx('orders').insert(data).returning('*');   // runs on the caller's transaction
}

await knex.transaction(async (trx) => {
  await createOrder(trx, { customer_id: 1, total: 100.00 });
});
```

If a helper accidentally calls the global `knex`, its statement runs **outside** the transaction — it commits even when the transaction later rolls back. That is the single most common transaction bug, and it is silent.

---

### 9. Express Integration

The controller (route handler) calls the query builder and never builds SQL strings itself. This keeps SQL in the Model layer, per MVC. [→ `../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md)

```js
// db/knex.js exports the shared instance (see §3)
const db = require('../db/knex');

// GET /posts/:id
exports.show = async (req, res, next) => {
  try {
    const post = await db('posts')
      .join('users', 'posts.author_id', 'users.id')
      .select('posts.*', 'users.name as author')
      .where('posts.id', req.params.id)
      .first();

    if (!post) return res.status(404).render('errors/404');
    res.render('posts/show', { post });
  } catch (err) {
    next(err);                                  // hand it to the Express error middleware
  }
};

// POST /posts
exports.create = async (req, res, next) => {
  try {
    const [post] = await db('posts')
      .insert({
        title: req.body.title,                  // Knex binds these as $1, $2, …
        body: req.body.body,
        author_id: req.session.userId,          // ← never trust a client-sent author_id
      })
      .returning('*');
    res.redirect(`/posts/${post.id}`);
  } catch (err) {
    next(err);
  }
};
```

Two disciplines are load-bearing:

1. **`async` handlers always wrap awaits in `try/catch` and call `next(err)`.** A rejected promise in an Express 4 handler that nobody catches becomes an unhandled rejection that can crash the process. (Express 5 forwards rejected promises, but the course targets Express 4.)
2. **`author_id` comes from the session, not the request body.** Never let the client choose an identity value; that is a broken-object-level authorization bug, not a formatting one. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)

---

### 10. Command / Code Cheat Sheet

| Construct | What it does | When to use | Example |
|-----------|--------------|-------------|---------|
| `knex('t').insert({...}).returning('*')` | Insert and get the created row | Creating a record and needing its id/timestamps | `const [p] = await knex('posts').insert(data).returning('*')` |
| `knex('t').select('id','title')` | Read chosen columns | Avoid `SELECT *` in app queries | `await knex('posts').select('id','title')` |
| `knex('t').where({ id })` | Filter with bound parameters | Every filtered read/update/delete | `await knex('posts').where({ published: true })` |
| `knex('t').where({ id }).update({...}).returning('*')` | Update matching rows, get them back | Any edit; **never omit `.where`** | `await knex('posts').where({id}).update({title}).returning('*')` |
| `knex('t').where({ id }).del()` | Delete matching rows | Removing a record; returns count | `await knex('posts').where({ id }).del()` |
| `.first()` / `.pluck('title')` | One row object / one column as array | You want a single row or a flat list | `await knex('posts').where({id}).first()` |
| `knex('t').count('* as n').groupBy('author_id')` | Aggregate + group | Dashboards, "posts per author" | `await knex('posts').select('author_id').count('* as n').groupBy('author_id')` |
| `knex('a').join('b', 'a.fk', 'b.id')` | Inner join | Combine related tables (only matches) | `knex('posts').join('users','posts.author_id','users.id')` |
| `knex.transaction(async trx => {...})` | Atomic multi-statement unit | Order → stock → payment flows | see §8 |
| `knex('t').whereRaw('expr ?', [v])` | Custom SQL with bound value | PostgreSQL features Knex doesn't model | `.whereRaw('LOWER(title) LIKE ?', ['%x%'])` |
| `npx knex migrate:make name` | Scaffold a migration | Before changing schema | `npx knex migrate:make add_slug_to_posts` |
| `npx knex migrate:latest` / `rollback` / `status` | Apply / undo / inspect migrations | Set up or reset an environment | `npx knex migrate:latest` |
| `npx knex seed:run` | Load dev data | Fresh local DB, demos, CI fixtures | `npx knex seed:run` |

---

### 11. Practical Lab — Full CRUD Against Dockerized Postgres

This lab connects to the throwaway PostgreSQL from the engine note, applies the migrations and seed above, then prints every CRUD result so you can verify each step.

**Step 1 — start the database** (from [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)):

```bash
docker run -d --name pg16 -p 5432:5432 \
  -e POSTGRES_USER=shopuser -e POSTGRES_PASSWORD=shoppass -e POSTGRES_DB=shop \
  postgres:16
```

**Step 2 — project layout:**

```
project/
├─ .env                 # DATABASE_URL=postgres://shopuser:shoppass@localhost:5432/shop
├─ knexfile.js
├─ db/
│  ├─ knex.js           # shared instance
│  ├─ migrations/       # 20261010120000_create_users.js, 20261010120100_create_posts.js
│  └─ seeds/            # 01_users.js, 02_posts.js
└─ lab.js
```

**Step 3 — build the schema and load data:**

```bash
npx knex migrate:latest
npx knex seed:run
```

```
Batch 1 run: 2 migrations
Batch 1 run: 2 seeds
```

**Step 4 — `lab.js`:**

```js
const knex = require('knex');
const config = require('./knexfile.js');
const db = knex(config.development);

(async () => {
  try {
    console.log('READ all:');
    console.log(await db('posts').select('id', 'title', 'published'));

    console.log('READ one (first):');
    console.log(await db('posts').where({ id: 3 }).first());

    console.log('JOIN posts + authors:');
    console.log(await db('posts')
      .join('users', 'posts.author_id', 'users.id')
      .select('posts.title', 'users.name as author')
      .orderBy('posts.id'));

    console.log('CREATE:');
    const [created] = await db('posts')
      .insert({ title: 'Created in lab', body: 'x', author_id: 2 })
      .returning('*');
    console.log(created);

    console.log('UPDATE:');
    const [updated] = await db('posts')
      .where({ id: created.id })
      .update({ published: true })
      .returning('*');
    console.log(updated);

    console.log('DELETE:');
    const removed = await db('posts').where({ id: created.id }).del();
    console.log(`rows deleted: ${removed}`);

    console.log('COUNT published:');
    const [{ count }] = await db('posts').where({ published: true }).count('*');
    console.log(`published = ${count} (type: ${typeof count})`);
  } catch (err) {
    console.error(err);
  } finally {
    await db.destroy();               // close the pool so Node exits
  }
})();
```

Run it:

```bash
node lab.js
```

**Expected output** (`id`s and timestamps vary; shapes do not):

```
READ all:
[
  { id: 1, title: 'Hello world', published: true },
  { id: 2, title: 'Draft notes', published: false },
  { id: 3, title: 'SQL joins', published: true }
]
READ one (first):
{ id: 3, title: 'SQL joins', body: 'Inner vs outer', published: true, author_id: 2, ... }
JOIN posts + authors:
[
  { title: 'Hello world', author: 'Ana' },
  { title: 'Draft notes', author: 'Ana' },
  { title: 'SQL joins', author: 'Binh' }
]
CREATE:
{ id: 4, title: 'Created in lab', body: 'x', published: false, author_id: 2, created_at: 2026-10-10T..., updated_at: 2026-10-10T... }
UPDATE:
{ id: 4, title: 'Created in lab', body: 'x', published: true, author_id: 2, ... }
DELETE:
rows deleted: 1
COUNT published:
published = 3 (type: string)
```

**What to notice:** the `count` came back as the **string** `'3'`, not the number `3` — PostgreSQL returns `bigint` as a string to avoid precision loss, and Knex passes it through. `typeof count === 'string'` is the gotcha in the next section. Also note that the UPDATE changed only row 4, because it carried a `.where({ id })`; deleting without it would have emptied the table.

```bash
docker rm -f pg16    # destroy the whole database
```

---

### 12. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `update`/`del` with no `.where()` | **Every row changes** — and the query succeeds | Always build `.where(...)` before `.update()`/`.del()`; review the chain end-to-end |
| Missing `.returning('*')` on PostgreSQL | `insert` gives a count, not the row; then a second `SELECT` to find the id | Add `.returning('*')` (or the specific columns) to `insert`/`update`/`del` |
| `count()` returned as a string | `"3" + 1 === "31"` in JS | `Number(count)` / `parseInt(count, 10)`, or `count('*::int as n')` to cast server-side |
| Forgetting `.first()` | You keep indexing `[0]` and crashing on empty results | `.first()` returns the object or `undefined`; handle the empty case explicitly |
| Statement outside the transaction | A helper used the global `knex`, so it commits even after rollback | Pass `trx` into every helper and call `.transacting(trx)` / use `trx(...)` |
| Connection pool exhaustion | Requests hang; `Knex: Timeout acquiring a connection` | Use one shared instance; cap `pool: { max }`; never hold a transaction open across an `await` of I/O; always `db.destroy()` in scripts |
| `whereRaw` with string interpolation | SQL injection survives the "safe" builder | Always pass values as the second argument: `.whereRaw('… ?', [v])` |
| Random `knex(...)` per request | Pools multiply; the server hits `too many clients already` | Export one instance from `db/knex.js` and `require` it everywhere |
| Editing an applied migration | The change never runs on any machine that already applied it | Add a new migration instead; treat applied migrations as immutable history |
| `SELECT *` everywhere | Breaks when a column is renamed/added; ships large `text` columns | Select only the columns the caller needs |
| Floating point for money via `decimal` | `table.float('price')` yields cents errors | `table.decimal('price', 12, 2)` → `numeric(12,2)` |

---

### 13. Comparison Table — Knex vs the Alternatives

| Dimension | Knex (query builder) | Raw `pg` + SQL strings | Full ORM (Prisma / Sequelize) |
|-----------|----------------------|------------------------|-------------------------------|
| Abstraction level | Thin: builds SQL, you still think in SQL | None | High: models, migrations, relations, often a schema DSL |
| Injection safety | Parameter binding by construction | Manual `$1` discipline on every query | Bound queries generated internally |
| Visibility of SQL | High (call `.toString()`) | Highest | Low (generated; can surprise) |
| Migrations | Built in (`knex migrate`) | You build/own the tooling | Built in, often more opinionated |
| Learning cost | Low if you know SQL | Zero abstraction, but every query is hand-written | Higher: learn the ORM's rules |
| Best for | The course's SSR CRUD app; teams that want SQL visible | Scripts, migrations, one-off admin queries | Large apps where boilerplate reduction wins |
| Risk | You must still know SQL to use it well | Injection/typo risk without care | ORM leaks / N+1 queries / escaping the abstraction |

Knex sits deliberately in the middle: it removes string-concatenation bugs and adds migrations, without hiding the SQL you are expected to understand.

---

## 🧪 Practice Tasks

> No solutions provided. Write them, run them, then ask for a graded review or a harder variant.

- [ ] **Task 1 — Scaffold from scratch.** Create a `knexfile.js`, two migrations (`authors`, `books` with a FK and an index on the FK column), and matching seeds. Run `migrate:latest` and `seed:run` on a fresh Docker Postgres, then `migrate:rollback` twice and confirm the tables are gone.
- [ ] **Task 2 — CRUD controller.** Build an Express router with `GET /books`, `GET /books/:id`, `POST /books`, `PUT /books/:id`, `DELETE /books/:id`. Every handler must use the shared Knex instance and `try/catch → next(err)`. Verify each endpoint shows the new row via `.returning('*')`.
- [ ] **Task 3 — Prove the scope trap (safely).** On a table with ~5 rows, write a script that first logs `.update(...).toSQL()` **without** a `.where()`, then deletes and re-seeds, then runs the correct scoped update. Report the exact SQL of the dangerous statement and explain why it "succeeds".
- [ ] **Task 4 — Transaction under failure.** Write a `knex.transaction` that inserts an order, decrements stock, then deliberately throws. Confirm (by querying after the throw) that neither the order nor the stock change persisted. Then change one helper to use the global `knex` instead of `trx`, and report the new, wrong behaviour.

---

## 🔗 Tech Cross-References

- **PostgreSQL core — every Knex call is SQL underneath.** `RETURNING`, `$1` binding, transactions, and the FK-index rule are engine features this builder exposes. [→ `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)
- **Security — parameter binding is the injection defence.** Knex's `.where({...})` compiles to `$1` placeholders; the wire protocol sends query and values separately, so values cannot be re-parsed as SQL. This is why the "safe builder" claim holds. [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)
- **Git & GitHub — migrations mirror version control.** An ordered, reviewable, reversible history of schema changes; applied migrations are immutable, exactly like pushed commits. [→ `../../git_github/README.md`](../../git_github/README.md)
- **Docker — the database this lab talks to.** `docker run postgres` plus a `DATABASE_URL` gives the disposable engine the migrations run against. [→ `../docker/README.md`](../docker/README.md)
- **Express MVC — the layer that calls the builder.** The route handler is the controller; Knex lives in the Model; templates in the View. [→ `../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md)

---

## 📝 Research Journey

- **Why:** I kept writing SQL as strings inside my route handlers and it felt fine — until I saw how easy it was to concatenate one user value and open an injection hole, and until a teammate ran my app on a fresh machine and the tables simply did not exist. Knex showed up in the course as "the database library", and I wanted to understand it as a *tool*, not as magic: what SQL it emits, why migrations matter, and where the sharp edges are.
- **Struggle:** The scope trap nearly got me. I wrote `knex('posts').update({ published: true })` while refactoring, ran it, and every post was published — with no error. That moment made me treat `.where()` as the first thing I write, not the last. The second struggle was transactions: my helper quietly called the global `knex` instead of the `trx` I passed in, so a rolled-back order still left its side-effect committed. The `.transacting(trx)` rule is now muscle memory.
- **Aha moment:** Appending `.toString()` to a builder and seeing my JavaScript chain become the exact SQL from the engine note. The layers snapped together: Knex is a *compiler* to SQL with `$1` holes, `pg` is the driver, and PostgreSQL does the real work. Migrations as "Git for the schema" was the other one — the `knex_migrations` table is just a commit log, and `rollback` is `revert`.
- **Career link:** The course grades this, but more importantly every backend I read for the rest of my career — every "orders", "ledgers", "feature store" service — is CRUD plus migrations plus transactions. When I later build ML data pipelines, the ingestion job that writes features is Knex (or an equivalent) underneath, and a non-atomic ingest is exactly how you get training-time/serving-time skew. Understanding the query builder is understanding the layer where product code meets data.

## 🔗 Related Topics

- [`07-postgresql-full-text-search.md`](07-postgresql-full-text-search.md) — the search feature built on top of this CRUD layer, using `.whereRaw('@@')`
- [`08-db-bottlenecks-indexing-and-caching.md`](08-db-bottlenecks-indexing-and-caching.md) — where indexes and pooling (the pitfall above) stop being enough
- [`01-relational-model-and-sql-basics.md`](01-relational-model-and-sql-basics.md) — the joins and `SELECT` this builder compiles to
- [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) — `RETURNING`, `$1`, transactions, and indexes at the engine level
- [`../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md) — the controller calling these queries
- [`../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md) — `limit`/`offset` and search used by real features
- [`README.md`](README.md) — folder roadmap (relational model → engine → Knex → search → bottlenecks)
- 🔗 [Joblink](https://github.com/qminh77/joblink) — a real full-stack repo to read against these patterns
- [SUGGESTED] `10-migrations-and-schema-evolution.md` — this note treats migrations as a tool; a dedicated note on evolving a *live* schema (zero-downtime `ALTER`, backfills, destructive-change playbooks) is the missing next step.
- [SUGGESTED] `bridge-knex-and-postgrest.md` — Knex for the app's write path vs PostgREST's generated REST layer; a bridge note would clarify which layer owns which query and how RLS changes the story.

## 🤔 Open Questions

- [ ] When does a query builder stop paying for itself and I should move to an ORM — or is the honest answer "rarely, if you understand SQL"?
- [ ] What is the correct pool size for a Node server behind a load balancer, and how do I measure it instead of guessing `max: 10`?
- [ ] In the course's transaction example, is the callback form sufficient, or do I need explicit `SELECT … FOR UPDATE` for the stock decrement (ties back to `databases/02`)?
- [ ] Do I need a separate read-only connection/pool for heavy reporting queries so they do not starve the write pool?
- [ ] How do I test migrations — run `latest`/`rollback` in CI against a throwaway Postgres on every PR, and what does that look like with GitHub Actions? [→ `../../git_github/README.md`](../../git_github/README.md)

---
*Know the SQL your builder emits. A query builder you cannot read is just an ORM you cannot debug.*

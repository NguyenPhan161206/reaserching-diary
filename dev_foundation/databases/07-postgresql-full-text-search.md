# PostgreSQL Full-Text Search — `tsvector`, `tsquery` & Ranking

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#PostgreSQL`, `#Search`, `#FullTextSearch`, `#GIN`, `#tsvector`, `#SQL`
**References:** PostgreSQL documentation — "Full Text Search" (§12) and "Text Search Types/Functions/Indexes" (§12.2, 12.3, 12.9); `postgresql.org`; `pg_trgm` contrib module docs; the course syllabus §4.5 (full-text search)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) (indexes, `EXPLAIN`, GIN); [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) (feeding these queries from the app)

---

## 📌 Key Takeaways (TL;DR)

- **Full-text search is language-aware *term* search, not substring matching.** SQL's `LIKE '%term%'` matches character runs and cannot use a normal index; full-text search parses text into **lexemes** (stemmed words), drops stop words, and matches *terms*.
- **The two data types and one operator:** a **`tsvector`** is a parsed document (an inner list of lexemes with positions); a **`tsquery`** is a parsed query (terms combined with `&`, `|`, `!`, `<->`); the **`@@`** operator returns "document matches query".
- **Use `plainto_tsquery` (or `websearch_to_tsquery`) for end-user input.** `to_tsquery` requires the user to write correct syntax and **throws** on stray punctuation; the `plain`/`websearch` variants parse raw text safely.
- **Ranking is what makes it a search feature.** `ts_rank` scores matches; `setweight(..., 'A')` on the title and `'B'` on the body makes a title hit outrank a body hit — the difference between "search" and "list of rows that contain the word".
- **Without a GIN index you get a sequential scan.** Store the `tsvector` (a generated column or trigger) and `CREATE INDEX ... USING GIN (search_vector)`. Verify with `EXPLAIN`.
- **This is the lexical half of RAG.** Term matching with ranking is the same family as the BM25 retrievers behind modern retrieval-augmented generation. [→ `../../ai_ml/03-rag-and-learning-to-rank.md`](../../ai_ml/03-rag-and-learning-to-rank.md)

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 Why `LIKE '%term%'` is not search

`SELECT * FROM articles WHERE body LIKE '%postgres%'` does three things you do not want in a search box:

1. It matches **substrings**, not words: `'%postgres%'` matches `postgresql`, `postgres`, and also `notpostgres-quite`. There is no concept of a word boundary.
2. It **cannot use a standard B-tree index** when the pattern starts with `%`. PostgreSQL must read every row — a sequential scan, $\Theta(n)$.
3. It has **no ranking**. Every matching row is equally good; there is no way to say "title matches beat body matches".

A real search needs three things `LIKE` does not provide: **word identity** (stemming), **relevance** (ranking), and **an indexable representation**. PostgreSQL's full-text search provides all three.

#### 1.2 `tsvector` vs `tsquery` — document and query

- A **`tsvector`** is the *parsed document*. It is a sorted list of **lexemes** — normalised word roots — each with the positions where it appeared:

  ```sql
  SELECT to_tsvector('english', 'The quick brown foxes jumped over the lazy dogs');
  ```

  ```
  'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
  ```

  Read that output: `the` and `over` are **stop words** and were dropped; `foxes → fox`, `jumped → jump`, `lazy → lazi` are **stemmed**; the numbers are word positions. The document is now a compact, language-normalised "inner list" that can be indexed.

- A **`tsquery`** is the *parsed query* — lexemes combined with operators:

  ```sql
  SELECT to_tsquery('english', 'foxes & jump');   -- → 'fox' & 'jump'
  SELECT to_tsquery('english', 'fox | dog');      -- → 'fox' | 'dog'
  SELECT to_tsquery('english', 'fox & !dog');     -- → 'fox' & !'dog'
  ```

- The **`@@`** operator asks the question: *does this document match this query?*

  ```sql
  SELECT to_tsvector('english', 'foxes jumped') @@ to_tsquery('english', 'jump');
  -- → true
  ```

**Why it matters:** the whole feature is these three pieces. `to_tsvector` normalises the *document* once; `to_tsquery` normalises the *query* once; `@@` compares them. Every other function (`ts_rank`, `ts_headline`, `setweight`) either scores or decorates that comparison.

#### 1.3 Stemming and stop words are what make it "language-aware"

The `'english'` **configuration** bundles a **stemmer** (Porter-style: `foxes → fox`) and a **stop-word list** (`the`, `over`, `and` …). The `'simple'` configuration does *neither* — it only lowercases and tokenises. The right choice depends on the data, not on habit (see §4.3 for the config, and the pitfalls table in §11).

---

### 2. Mechanism (How It Works)

#### 2.1 What `to_tsvector` and `to_tsquery` actually do

Parsing text into a `tsvector` runs a small pipeline:

```
 raw text
   │
   ▼
 1. TOKENISE   split on non-word boundaries        "foxes jumped"
   │
   ▼
 2. NORMALISE  lowercase                            "foxes" "jumped"
   │
   ▼
 3. DICTIONARY apply stop-word list + stemmer       drop stop words; "foxes"→"fox", "jumped"→"jump"
   │
   ▼
 4. POSITIONS  record where each lexeme occurred     'fox':4 'jump':5
   │
   ▼
 tsvector (sorted lexeme list, with positions & weights)
```

`to_tsquery` (and its safe siblings) run the **same normalisation** on the query text, then combine the resulting lexemes into a small **expression tree** using the operators the query contains (`&`, `|`, `!`, `<->`). The `@@` operator then evaluates that tree against the document's lexeme-and-position list. Because **positions are stored**, proximity operators like `<->` (adjacent) and `<2>` (within 2) can be checked — that is why phrase search works at all.

#### 2.2 How a GIN index answers `@@`

A **GIN** index is an **inverted index**: a map from each lexeme to the set of rows that contain it (its *posting list*).

```
         inverted index (GIN)
 'postgr'  →  rows {1, 2, 7}
 'search'  →  rows {2, 5, 7}
 'deep'    →  rows {2}

 query 'postgr' & 'search'
   lookup both posting lists → intersect → {2, 7}
```

The execution has two phases, which is exactly what `EXPLAIN` shows:

1. **Bitmap Index Scan** — look up the query's lexemes in the index and combine the posting lists with the query operators (`&` = intersect, `|` = union, `!` = subtract). This yields a *candidate* set of row locations.
2. **Bitmap Heap Scan + `Recheck Cond`** — fetch the candidate rows from the heap and re-test `@@` exactly, because a GIN index can produce false positives on complex expressions and because phrase/proximity conditions need the stored positions.

**Why it matters:** this is the whole performance story. Without the index, `@@` must **parse every row's text on every query** — a sequential scan, $\Theta(n)$ per search. With the GIN index it becomes "look up a handful of posting lists and intersect them" — the same idea as the index at the back of a book. It also explains the trade-off: every `INSERT`/`UPDATE` must update the inverted index, so writes get slower and the index costs disk.

---

### 3. The Three Levels of "Search" in PostgreSQL

Search is a spectrum. Know which level the requirement actually needs.

#### Level A — Substring: `LIKE` / `ILIKE`

```sql
SELECT * FROM articles WHERE body ILIKE '%postgres%';
```

Accurate for exact substrings, but as noted: no word boundaries, no ranking, and `%…%` cannot use a B-tree index. Fine for small tables and admin filters; a trap at scale.

#### Level B — Fuzzy substring: `pg_trgm` (trigram GIN)

`pg_trgm` breaks text into **trigrams** — every 3-character sliding window — and builds a GIN index over them. This buys you **typo tolerance** and makes `ILIKE '%…%'` indexable.

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX articles_title_trgm ON articles USING GIN (title gin_trgm_ops);

SELECT similarity('postgres', 'postgre');   -- → ~0.7, a fuzzy similarity score
SELECT title FROM articles WHERE title % 'postgre';        -- similarity above threshold
SELECT title FROM articles WHERE title ILIKE '%postgre%';  -- now uses the trigram index
```

Use trigram when the requirement is "find things that look like this string" — autocomplete, typo fixing, name matching. It has no stemming and no relevance model beyond string similarity.

#### Level C — True full-text: `tsvector` / `tsquery` + GIN

This is the level for "search this corpus for these words and rank the results". Word matching, stemming, stop words, ranking, highlighting.

#### The comparison table

| | `LIKE` / `ILIKE` | `pg_trgm` | Full-text (`tsvector`/`tsquery`) |
|---|---|---|---|
| Matches | character substring | fuzzy substring (trigrams) | **words / lexemes** |
| Word boundaries | no | no | yes |
| Stemming / stop words | no | no | yes (per language config) |
| Typo tolerance | no | **yes** (`%`, `similarity`) | no (needs a different tool) |
| Ranking | no | similarity score | **`ts_rank`** with weights |
| Indexable `%…%` | no | yes (GIN `gin_trgm_ops`) | n/a |
| Index | B-tree (prefix only) | GIN | GIN on `tsvector` |
| Best for | small admin filters | autocomplete, misspellings | search box over documents |

**Decision rule:** exact substring → `LIKE`; "looks like" / typo-tolerant → `pg_trgm`; "find documents about X, best first" → full-text. Large apps combine Level B and C.

---

### 4. Building Blocks

#### 3.1 The query constructors

```sql
-- to_tsquery: authoritative syntax; the user must write & | ! <-> themselves
SELECT to_tsquery('english', 'postgres & search');       -- → 'postgr' & 'search'

-- plainto_tsquery: take a raw string, AND the surviving terms. SAFE for user input.
SELECT plainto_tsquery('english', 'the postgres search'); -- → 'postgr' & 'search'
--   (no operators to get wrong; punctuation is ignored, not parsed as syntax)

-- websearch_to_tsquery: accept web-style syntax, still parse safely
SELECT websearch_to_tsquery('english', '"postgres search" -mysql');
-- → 'postgr' <-> 'search' & !'mysql'
```

| Constructor | User writes | Safety | Use when |
|---|---|---|---|
| `to_tsquery` | `term & term`, `!term`, `a \| b` | **unsafe** — throws on stray punctuation | you control the query text (code, not a user) |
| `plainto_tsquery` | `term term` | safe — ignores operators | a simple search box |
| `websearch_to_tsquery` | `"phrase" -exclude OR` | safe — parses web syntax | a search box with quote/exclusion support |

**Why it matters:** `to_tsquery('english', userInput)` throws `syntax error in tsquery` the moment a user types `C++` or `rock & roll`. Never feed untrusted text to `to_tsquery`. `plainto_tsquery` treats `C++` as an ordinary token and moves on.

#### 3.2 The `@@` operator and language configs

```sql
SELECT to_tsvector('english', 'The running fox') @@ plainto_tsquery('english', 'running');
-- → true   (run == run after stemming)
```

The **config** (`'english'`, `'simple'`, …) must be the *same* on both sides, or the lexemes will not line up.

#### 3.3 `'english'` vs `'simple'` — choose per column

```sql
SELECT to_tsvector('english', 'running dogs');  -- → 'dog':2 'run':1     (stemmed)
SELECT to_tsvector('simple',  'running dogs');  -- → 'dogs':2 'running':1 (lowercased only)
```

| Config | Stems? | Drops stop words? | Use for |
|--------|--------|-------------------|---------|
| `english` | yes | yes | English prose documents |
| `simple` | no | no | proper nouns, codes, identifiers, **non-English text** |
| other languages | per language | per language | the matching natural language |

**Why it matters:** the English stemmer will mangle a language it does not know. For Vietnamese or mixed code/text, `simple` (or a language-specific config) is the honest choice — an incorrect stemmer produces lexemes that never match the query.

---

### 5. Phrase & Prefix Search

#### 4.1 Phrase search with proximity operators

```sql
-- exact adjacent phrase
SELECT to_tsquery('english', 'quick <-> fox');   -- → 'quick' <-> 'fox'
-- within N positions (here: at most 2 apart)
SELECT to_tsquery('english', 'quick <2> fox');   -- → 'quick' <2> 'fox'
```

```sql
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'full <-> text');
```

Phrase operators are what `websearch_to_tsquery` produces from quoted input (`'"full text"'`), and they are how you stop "full" and "text" appearing far apart from counting as the same phrase.

#### 4.2 Prefix search — `term:*`

```sql
SELECT to_tsquery('english', 'postgr:*');   -- → 'postgr':*
```

```sql
SELECT title FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgre:*');
```

This is the "search as you type" behaviour: `postgre` matches `postgres`, `postgresql`, `postgresql-tutorial`. Three caveats:

1. **It needs special handling for user input.** `to_tsquery` requires the `:*` you append, and you must first strip any operator characters the user typed — otherwise `C++:*` throws. The safe pattern is to extract the last word, sanitise it to letters/digits, then build the prefix query.
2. **A prefix GIN index helps but is not free.** `gin_trgm_ops` or `text_pattern_ops` can accelerate prefix matching; a plain full-text GIN index supports `:*` too but scans more of the index.
3. **Do not prefix a stop word** — `to_tsvector` dropped it, so there is nothing to match against.

---

### 6. Ranking — Turning Matches into a Result List

#### 5.1 `ts_rank` and `ts_rank_cd`

```sql
SELECT id, title,
       ts_rank(search_vector, query) AS rank
FROM articles, plainto_tsquery('english', 'postgres search') AS query
WHERE search_vector @@ query
ORDER BY rank DESC NULLS LAST;
```

- `ts_rank` scores by term frequency and how recently the term appeared in the document.
- `ts_rank_cd` ("cover density") also considers how *close* the query terms are to each other — better when proximity implies relevance.

Both return a `real` (float). Values are not comparable across different queries; they are only meaningful for ordering *within one result set*.

#### 5.2 Weighting: title outranks body

Relevance is a product decision: a hit in the title usually matters more than a hit in the body. `setweight` labels lexeme positions `A` (best) through `D` (worst); `ts_rank` uses those labels.

```sql
UPDATE articles SET search_vector =
  setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
  setweight(to_tsvector('english', coalesce(body,  '')), 'B');
```

The `||` **concatenates** the two weighted vectors into one. Now a title match carries weight `A` and a body match weight `B`. The default weight array is `{D, C, B, A} = {0.1, 0.2, 0.4, 1.0}`, so `A` is worth 2.5× `B`. You can override it in the rank call:

```sql
-- weight array order is {D, C, B, A}
ts_rank('{0.1, 0.2, 0.4, 1.0}', search_vector, query)
```

`coalesce(..., '')` is not optional: `to_tsvector` on a `NULL` returns `NULL`, and `NULL || vector` poisons the whole concatenation back to `NULL`.

#### 5.3 Ordering with ties and nulls

```sql
ORDER BY rank DESC NULLS LAST;
```

`NULLS LAST` guarantees rows with no rank (if the query joins leftward) sink to the bottom rather than jumping to the top — PostgreSQL otherwise treats `NULL` as larger than everything in `DESC`.

---

### 7. Indexing a `tsvector`

#### 6.1 The GIN index

```sql
-- expression index: index the vector computed from the columns
CREATE INDEX articles_search_idx
  ON articles USING GIN (to_tsvector('english', coalesce(title,'') || ' ' || coalesce(body,'')));
```

A **GIN** (Generalized Inverted Index) is the right structure for `tsvector`: it maps each lexeme to the set of rows containing it — the **inverted index**, the same idea behind a book's index and behind Lucene/Elasticsearch. A B-tree cannot answer `@@` (it indexes whole values, not the elements inside a `tsvector`).

**Without it, `@@` forces a sequential scan.** Check with `EXPLAIN`:

```
-- no index:
Seq Scan on articles  (cost=0.00..… rows=…) (actual time=… rows=…)
  Filter: (search_vector @@ ...)

-- with GIN:
Bitmap Heap Scan on articles  (cost=…) (actual time=0.05..0.12 rows=2)
  Recheck Cond: (search_vector @@ ...)
  ->  Bitmap Index Scan on articles_search_idx
```

#### 6.2 Keep the vector in sync: generated column vs trigger

You do **not** want to recompute `to_tsvector` in every query *and* index, but you also do not want to hand-maintain a column. Two idiomatic options:

**Generated column (PostgreSQL 12+, preferred):**

```sql
ALTER TABLE articles ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title,'')), 'A') ||
    setweight(to_tsvector('english', coalesce(body, '')), 'B')
  ) STORED;

CREATE INDEX articles_search_idx ON articles USING GIN (search_vector);
```

The database recomputes the column on every `INSERT`/`UPDATE` — always consistent. **Critical gotcha:** a generated column requires an **immutable** expression. `to_tsvector('english', ...)` with the config passed *explicitly* is immutable; bare `to_tsvector(col)` uses a session default and is only *stable* — it will be rejected.

**Trigger (needed when the expression is not immutable, or on older PostgreSQL):** a `BEFORE INSERT OR UPDATE` trigger reassigns `NEW.search_vector` from `NEW.title`/`NEW.body`. More moving parts, but works when a generated expression cannot.

If you cannot store the vector at all, an **expression GIN index** over the exact `to_tsvector(...)` expression also works — but the query must use the **identical** expression, or the planner will not match the index and you are back to a seq scan.

---

### 8. Integration — A Search Route

The route takes user text, converts it safely with `plainto_tsquery`, filters with `@@`, ranks, and renders. This plugs into the Express/Knex layer from `databases/06` and the pagination/search feature in `web_server/08`. [→ `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md), [→ `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)

```js
// GET /search?q=postgres%20search
const db = require('../db/knex');

exports.search = async (req, res, next) => {
  try {
    const q = (req.query.q || '').trim();
    if (!q) return res.render('search/results', { q, results: [] });

    const results = await db('articles')
      .select(
        'id',
        'title',
        db.raw(
          `ts_headline('english', body,
             plainto_tsquery('english', ?),
             'StartSel=<mark>, StopSel=</mark>, MaxWords=35, MinWords=15') AS snippet`,
          [q]
        )
      )
      .whereRaw(`search_vector @@ plainto_tsquery('english', ?)`, [q])
      .orderByRaw(`ts_rank(search_vector, plainto_tsquery('english', ?)) DESC NULLS LAST`, [q])
      .limit(20);

    res.render('search/results', { q, results });
  } catch (err) {
    next(err);
  }
};
```

Three things to internalise:

1. **`plainto_tsquery` on user input, always.** It cannot be tricked into a syntax error and it cannot be an injection vector — the value is still bound as `$1`, and the function only *parses* it.
2. **`ts_headline` builds the highlighted snippet** straight from the document, wrapping matched lexemes in your tags. Render the snippet as raw HTML in the template (it is generated server-side, not user input).
3. **The index needs the query to use the same vector expression.** Here `search_vector` is the stored/generated column, so `@@ search_vector` matches the GIN index exactly.

---

### 9. Command / Code Cheat Sheet

| Construct | What it does | When to use | Example |
|-----------|--------------|-------------|---------|
| `to_tsvector('english', text)` | Parse a document into lexemes | Building the searchable representation | `to_tsvector('english', title)` |
| `to_tsquery('english', 'a & b')` | Parse an explicit query (operators) | Internal/code-built queries only | `to_tsquery('english', 'fox & jump')` |
| `plainto_tsquery('english', input)` | Parse raw text, AND the terms | **End-user search box** | `plainto_tsquery('english', $1)` |
| `websearch_to_tsquery('english', input)` | Parse web syntax safely | Search box with `"quotes"` / `-excludes` | `websearch_to_tsquery('english', '"a b" -c')` |
| `vector @@ query` | Test document-matches-query | The `WHERE` clause of every search | `search_vector @@ plainto_tsquery(...)` |
| `ts_rank(vector, query)` | Relevance score | `ORDER BY` the results | `ts_rank(search_vector, query) DESC` |
| `setweight(vector, 'A')` | Label lexemes A–D for weighting | Boost title over body | `setweight(to_tsvector(...,title),'A')` |
| `CREATE INDEX ... USING GIN (vector)` | Inverted index for `@@` | Any real search corpus | `CREATE INDEX ON articles USING GIN (search_vector)` |
| `ts_headline('english', body, query, opts)` | Highlighted snippet | Result previews | `ts_headline('english', body, query, 'StartSel=<mark>')` |
| `CREATE EXTENSION pg_trgm` + `gin_trgm_ops` | Trigram fuzzy search | Typo tolerance / `ILIKE` indexing | `CREATE INDEX ... USING GIN (title gin_trgm_ops)` |
| `similarity(a, b)` / `a % b` | Fuzzy similarity score / threshold | "Looks like" matching | `SELECT similarity('hello','helo')` |
| `EXPLAIN ANALYZE ...` | Show the plan | Prove the GIN index is used | `EXPLAIN ANALYZE SELECT ...` |

---

### 10. Practical Lab — Watch Title Weighting Change the Ranking

Dockerized PostgreSQL from [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md), then a table whose `search_vector` weights the title above the body. The goal: the same query returns a **different order** depending on where the matching words live.

**Step 1 — start the database:**

```bash
docker run -d --name pg16 -p 5432:5432 \
  -e POSTGRES_USER=shopuser -e POSTGRES_PASSWORD=shoppass -e POSTGRES_DB=shop \
  postgres:16
docker exec -it pg16 psql -U shopuser -d shop
```

**Step 2 — schema with a weighted generated column + GIN index:**

```sql
CREATE TABLE articles (
  id            SERIAL PRIMARY KEY,
  title         TEXT NOT NULL,
  body          TEXT NOT NULL,
  search_vector tsvector GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(body,  '')), 'B')
  ) STORED
);

CREATE INDEX articles_search_idx ON articles USING GIN (search_vector);
```

```
CREATE TABLE
CREATE INDEX
```

**Step 3 — insert rows where the SAME words appear in different places:**

```sql
INSERT INTO articles (title, body) VALUES
  ('Cooking pasta tonight',        'A short intro to postgres full text search'),
  ('PostgreSQL full text search',  'A deep dive into tsvector and tsquery'),
  ('Gardening in autumn',          'Plant bulbs in postgres-free soil this search season');
```

**Step 4 — query the same terms against every row, ranked:**

```sql
SELECT id, title,
       ts_rank(search_vector, plainto_tsquery('english', 'postgres search')) AS rank
FROM articles
WHERE search_vector @@ plainto_tsquery('english', 'postgres search')
ORDER BY rank DESC NULLS LAST;
```

**Expected output** (values are approximate — positions and lengths affect the exact float; the *order* is the point):

```
 id |            title            |   rank
----+-----------------------------+-----------
  2 | PostgreSQL full text search | 0.0607927
  1 | Cooking pasta tonight       | 0.0241396
  3 | Gardening in autumn         | 0.0205293
(3 rows)
```

**What to read in that output:**

- Row **2** wins because **both** `postgres` and `search` appear in its **title** (weight `A`).
- Row **1** has both words only in the **body** (weight `B`) — it still matches, but ranks lower.
- Row **3** matches, but the words are scattered in the body, so it ranks lowest.

**Step 5 — prove the weighting is doing the work.** Rebuild the vector *without* `setweight` and re-rank:

```sql
ALTER TABLE articles DROP COLUMN search_vector;
ALTER TABLE articles ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (
    to_tsvector('english', coalesce(title,'') || ' ' || coalesce(body,''))
  ) STORED;
CREATE INDEX articles_search_idx ON articles USING GIN (search_vector);
```

Re-run the query from Step 4. Row 2 still tends to lead, but the **gap collapses** — the title's `A` bonus is gone, so a title hit and a body hit are scored on frequency alone. That difference *is* the product decision you encoded.

**Step 6 — confirm the index is used:**

```sql
EXPLAIN ANALYZE
SELECT id FROM articles
WHERE search_vector @@ plainto_tsquery('english', 'postgres search');
```

```
Bitmap Heap Scan on articles  (cost=… rows=3) (actual time=0.05..0.08 rows=3 loops=1)
  Recheck Cond: (search_vector @@ plainto_tsquery('english', 'postgres'::text & 'search'::text))
  ->  Bitmap Index Scan on articles_search_idx  (actual time=0.02..0.02 rows=3 loops=1)
```

If you instead see `Seq Scan`, the query's expression does not match the indexed `search_vector` column — fix the mismatch, then re-check.

**Step 7 — a preview snippet:**

```sql
SELECT ts_headline('english', body, plainto_tsquery('english', 'postgres search'),
         'StartSel=<mark>, StopSel=</mark>') AS snippet
FROM articles WHERE id = 2;
```

```
A deep dive into tsvector and tsquery <mark>postgres</mark> matching <mark>search</mark> ...
```

```bash
docker rm -f pg16     # destroy the database
```

---

### 11. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Wrong language config (`'simple'` vs `'english'`) | Query and document produce different lexemes; nothing matches | Use the same config on both sides; pick `'simple'` for non-English/code text |
| Feeding user input to `to_tsquery` | `syntax error in tsquery` on `C++`, `&`, `:` | Use `plainto_tsquery` or `websearch_to_tsquery` for end-user input |
| Missing GIN index | `EXPLAIN` shows `Seq Scan` on a large table | `CREATE INDEX ... USING GIN (search_vector)` and verify with `EXPLAIN` |
| Index/query expression mismatch | `Seq Scan` even though an index exists | Ensure the query uses the exact indexed column/expression (e.g. `search_vector`) |
| `ILIKE '%x%'` at scale | Full table scan; latency grows linearly with rows | Trigram index (`pg_trgm` + `gin_trgm_ops`) if fuzzy, full-text if word search |
| `to_tsvector(NULL)` in a weighted vector | Whole `search_vector` becomes `NULL`; row stops matching | `coalesce(column, '')` before every `to_tsvector` in the vector expression |
| Generated column rejected | `generation expression is not immutable` | Pass config explicitly (`to_tsvector('english', …)`), not the session-default form |
| Prefix query built from raw input | Errors or no matches (`C++:*`, prefixing a stop word) | Sanitise the last token to alphanumerics, then append `:*` |
| Accents/diacritics not matched | `café` does not match `cafe` | `CREATE EXTENSION unaccent` and wrap input/columns; use `unaccent('dict', …)` form for immutability in indexes |
| Reading `ts_rank` values as absolute | Scores compared across different queries that look "wrong" | `ts_rank` is only an ordering signal within one query's result set |
| Forgetting `NULLS LAST` | Rank-less rows jump to the top on `DESC` | `ORDER BY rank DESC NULLS LAST` |

---

### 12. Comparison Table — `ts_rank` vs `ts_rank_cd`, and Full-Text vs the Alternatives

| Ranking function | Considers | Favours | Use when |
|------------------|-----------|---------|----------|
| `ts_rank` | term frequency, position, weights | documents that use the term more | general relevance |
| `ts_rank_cd` | the above + proximity ("cover density") | documents where query terms cluster together | phrase-like relevance |

| Approach | Handles typos | Understands words | Ranks | Scales with an index | Cost |
|----------|---------------|-------------------|-------|----------------------|------|
| `LIKE '%…%'` | no | no (substring) | no | no | cheap on tiny tables, O(n) always |
| `pg_trgm` | yes | no | similarity only | yes (GIN) | extra index size |
| Full-text `tsvector` | no | yes (stem/stop) | yes (`ts_rank`) | yes (GIN) | parse + storage + index |
| Dense vector search | yes (semantic) | meaning, not words | yes (cosine) | yes (ANN) | embeddings + a vector index |

The last row is the point of the career bridge: PostgreSQL's full-text search is the **lexical** retriever; embeddings are the **dense** retriever; real systems combine them. [→ `../../ai_ml/03-rag-and-learning-to-rank.md`](../../ai_ml/03-rag-and-learning-to-rank.md)

---

## 🧪 Practice Tasks

> No solutions provided. Do them, then ask for a graded review, a plan-reading check, or a harder variant.

- [ ] **Task 1 — Build and rank.** Create an `articles(title, body)` table with a weighted generated `search_vector` and a GIN index. Insert 10 rows with deliberately overlapping vocabulary. Run a `plainto_tsquery` search and explain, row by row, why each rank is what it is.
- [ ] **Task 2 — Prove the index matters.** From a clean database, run `EXPLAIN ANALYZE` on your search query *before* and *after* creating the GIN index with 100 000 rows loaded. Report both plans and the execution-time delta.
- [ ] **Task 3 — Safe prefix search.** Implement a `suggest(prefix)` query that matches `term:*`, taking a raw user string. Sanitise it so that inputs like `C++`, `a&b`, and `the` do not error and do not match a stop word. Show the exact strings that would break the naïve `to_tsquery` version.
- [ ] **Task 4 — Add typo tolerance.** `CREATE EXTENSION pg_trgm`, add a trigram GIN index on the title, and combine it with full-text in one query: return rows that either match the full-text query *or* have `title % query`. Explain when each branch fires and how you would order the merged results.
- [ ] **Task 5 (harder) — Keep the vector in sync without a generated column.** Use a `BEFORE INSERT OR UPDATE` trigger to maintain `search_vector`, then demonstrate that an update to `body` changes the ranking of a subsequent search.

---

## 🔗 Tech Cross-References

- **DSA — a GIN index is an inverted index.** It maps each lexeme to the set of rows that contain it, the same "value → locations" idea as the book index and hash-map-per-term; `ts_rank` is a scoring function layered on that structure. [→ `../dsa/05-data-structure-selection-guide.md`](../dsa/05-data-structure-selection-guide.md)
- **PostgreSQL core — indexes and `EXPLAIN` are the same machinery.** This note's GIN index and plan-reading are the engine note's index trade-off (faster reads, slower writes, more disk) applied to text. [→ `02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md)
- **AI/ML — this is the lexical half of RAG.** Term matching with ranking (BM25-style) is one retriever; embedding search is the other; hybrid systems fuse them. [→ `../../ai_ml/03-rag-and-learning-to-rank.md`](../../ai_ml/03-rag-and-learning-to-rank.md), [→ `../../ai_ml/02-search-systems-lexical-dense-hybrid.md`](../../ai_ml/02-search-systems-lexical-dense-hybrid.md)
- **Express / web server — the search route is a controller.** `GET /search?q=…` binds the query with `plainto_tsquery`, ranks, and renders highlighted snippets; this is the search feature of the course app. [→ `../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)
- **Knex — how the app emits these queries.** `.whereRaw('@@ plainto_tsquery(?, ?)', […])` and `.orderByRaw(...)` carry the feature into the query builder. [→ `06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md)

---

## 📝 Research Journey

- **Why:** The course asks for a search box, and my first instinct was `WHERE body LIKE '%' || $1 || '%'`. It "worked" on my ten test rows, which is exactly the danger — the failure only appears at real scale and only for real users who type `C++` or misspell a word. I wanted to know what search engines actually do, because the same question — "given a pile of text and a query, what is relevant?" — is the heart of the systems I want to build.
- **Struggle:** Two things. First, `to_tsquery` throwing a syntax error on ordinary punctuation felt unfair until I understood that it is a *parser* for a tiny query language, and users are not writing that language — hence `plainto_tsquery`. Second, the generated-column rejection (`generation expression is not immutable`) took a while: the config argument has to be explicit, because a function that depends on session state cannot be baked into stored data. That immutability rule is the same reason you cannot index `now()`.
- **Aha moment:** Watching the ranking flip when I moved the same two words from the body into the title. `setweight` is just four labels, but it is the difference between "rows that contain the word" and "the best answer first". The second aha was seeing a GIN index turn a `Seq Scan` into a `Bitmap Index Scan` — the inverted index from textbooks, live in PostgreSQL, the exact structure my DSA notes describe.
- **Career link:** Retrieval is the load-bearing wall of every serious AI product I want to build — RAG, semantic search, recommendations all start by *finding* the right documents. PostgreSQL full-text search is the lexical retriever, and understanding `tsvector`/`tsquery`/`ts_rank` means I understand the keyword half of hybrid search before I ever reach for an embedding model. It is also a lesson in not over-engineering: for a business app with 50 000 rows, this built-in, free, transactional search is very often the correct engineering choice over a separate search cluster.

## 🔗 Related Topics

- [`06-knex-query-builder-and-crud.md`](06-knex-query-builder-and-crud.md) — the query builder that emits these `@@` / `orderByRaw` statements
- [`02-postgresql-core-concepts.md`](02-postgresql-core-concepts.md) — indexes, `EXPLAIN`, and the GIN structure used here
- [`08-db-bottlenecks-indexing-and-caching.md`](08-db-bottlenecks-indexing-and-caching.md) — where text search, caching, and query cost meet under load
- [`../../ai_ml/03-rag-and-learning-to-rank.md`](../../ai_ml/03-rag-and-learning-to-rank.md) — the retrieval-and-ranking family this note belongs to
- [`../../ai_ml/02-search-systems-lexical-dense-hybrid.md`](../../ai_ml/02-search-systems-lexical-dense-hybrid.md) — BM25, dense, and hybrid search, of which `ts_rank` is the lexical cousin
- [`../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md) — the search feature that consumes this route
- [SUGGESTED] `bridge-postgres-fuzzy-search-and-embeddings.md` — a bridge note combining `pg_trgm` + full-text + `pgvector` in one hybrid query; it would connect this lexical note to the dense-retrieval notes in `ai_ml/`.
- [SUGGESTED] `09-jsonb-and-semi-structured-queries.md` — the other PostgreSQL feature the course app will reach for (flexible attributes), and its own GIN index story.

## 🤔 Open Questions

- [ ] When does the course's search feature outgrow built-in full-text and need Elasticsearch/Meilisearch — and is the honest answer "almost never for this app"?
- [ ] How do I combine `ts_rank` (lexical) with an embedding score (dense) in one query — reciprocal rank fusion in SQL, or fuse in the application layer? [→ `../../ai_ml/02-search-systems-lexical-dense-hybrid.md`](../../ai_ml/02-search-systems-lexical-dense-hybrid.md)
- [ ] What is the right behaviour for **non-English** (Vietnamese) content: `simple` config, a language-specific stemmer, or trigram matching only?
- [ ] Does a GIN index on a frequently-updated `search_vector` become a write bottleneck, and what does `fastupdate` change about that trade-off?
- [ ] How should snippet highlighting (`ts_headline`) handle user-supplied `StartSel`/`StopSel` without opening an XSS hole when the snippet is rendered as HTML? [→ `../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)

---
*`LIKE` finds characters. Full-text search finds words, ranks them, and finds them fast — that is the whole difference.*

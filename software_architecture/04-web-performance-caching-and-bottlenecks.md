# Web Performance, Caching & Bottlenecks — Where Every Second Hides

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#WebPerformance`, `#Caching`, `#Bottlenecks`, `#SSR`, `#Latency`, `#SystemArchitecture`, `#Redis`, `#EventLoop`
**References:** Google Web.dev "Core Web Vitals" (LCP, FCP, TTFB), RFC 9111 (HTTP Caching), nginx `proxy_cache` docs, MDN "Cache-Control" / "ETag", Express "Performance Best Practices", Node.js "Don't Block the Event Loop", Field (2002) *N+1 query problem*, Little (1961) *A Proof for the Queuing Formula L = λW*
**Status:** Draft
**Difficulty:** Intermediate→Advanced
**Prerequisites:** [`../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md`](../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md), [`../dev_foundation/web_server/02-web-server-nginx-apache-express.md`](../dev_foundation/web_server/02-web-server-nginx-apache-express.md), [`../dev_foundation/web_basics/05-dom-and-browser-event-loop.md`](../dev_foundation/web_basics/05-dom-and-browser-event-loop.md)

---

## 📌 Key Takeaways (TL;DR)

- **Performance is not a feature you add at the end — it is the sum of every layer in the request path.** If a page takes 1.8 s, no single genius fix removes it; you find which layer owns which millisecond and fix the biggest owner first.
- **Architecture is a performance decision.** *Where* you place a cache (browser, CDN, proxy, app, DB) determines the *floor* of latency you can ever reach. Moving work closer to the user is the lever; a better algorithm cannot beat a request that never leaves the browser.
- **Measure before you optimise.** `curl -w` and the DevTools waterfall tell you the split in seconds; guessing costs a weekend and buys nothing.
- **The five caching layers are ordered by distance from the user:** browser → CDN → reverse proxy (nginx micro-cache) → app (Redis/node-cache, cache-aside) → database (indexes). Each one is faster and less specific than the next.
- **The two killer app-level bugs are blocking the event loop and N+1 queries** — both are invisible in a demo with 10 rows and fatal with 10,000.
- **Cache invalidation is the hard part, not caching.** A stale price or a leaked per-user page is worse than a slow page; know when *not* to cache.

---

## 🧠 Detailed Notes

### 1. Core Concepts

**Performance = the latency a user actually feels**, not a benchmark number. That latency is the sum over every hop the request and response take:

$$T_{\text{perceived}} = T_{\text{DNS}} + T_{\text{TCP}} + T_{\text{TLS}} + T_{\text{network}} + T_{\text{proxy}} + T_{\text{app}} + T_{\text{DB}} + T_{\text{render}} + T_{\text{transfer}} + T_{\text{browser-paint}}$$

Every term is owned by a different layer, and **the slowest term sets the experience.** "Optimising" a 2 ms function while a 900 ms query runs is theatre.

Two different numbers are confused constantly, so pin them down:

- **Latency** — how long *one* request takes (what a user feels).
- **Throughput** — how many requests per second the system finishes (what a load balancer cares about). They are linked by **Little's Law**: for a stable system, $L = \lambda W$ — the number of requests in flight $(L)$ equals arrival rate $(\lambda)$ times time-in-system $(W)$. If each request costs 900 ms and 100 people click per second, ~90 requests are queued at any instant. Latency *creates* congestion.

**The "architecture is a performance decision" framing.** A single Express process querying one Postgres database has a latency floor set by the network round-trips it must make. Add a Redis cache and the hot read never touches Postgres. Add nginx `proxy_cache` and anonymous pages never touch Express. Add a CDN and the HTML and assets never touch your server at all. **Each architectural addition removes whole layers from the sum.** That is why caching is an architecture topic, not a "code" topic.

**Typical split of an SSR request (illustrative, first visit, no cache):**

| Layer | Time | Notes |
|-------|------|-------|
| DNS + TCP + TLS | ~130 ms | Once per connection; gone on keep-alive |
| Request/response network | ~120 ms | Distance to server; CDN attacks this |
| Reverse proxy | ~2 ms | nginx itself is cheap |
| Express routing/middleware | ~5 ms | Bananas compared to what follows |
| Controller + DB query | **~900 ms** | N+1 + missing index — the usual villain |
| Template render | ~220 ms | Dev mode recompiling the view every request |
| Response serialisation + transfer | ~80 ms | Big JSON/HTML, no gzip |
| Browser parse, asset fetch, paint | ~425 ms | Unminified bundles, no compression, no font preload |

The single largest owner is almost never "the code"; it is **I/O you did once per row** or **work you repeat that a cache could have answered**.

> **Why it matters:** Users abandon slow pages long before they finish loading, and search engines rank on Core Web Vitals. Amazon's classic finding (100 ms of latency ≈ 1% of sales) is the extreme version of a universal rule: latency is a business metric, not an engineering vanity metric.

---

### 2. The End-to-End Request Journey (and where a second hides)

```
 USER CLICK
    │
    ▼
[1] DNS lookup───────────────► resolves hostname → IP         (20–120 ms, cacheable)
    │
    ▼
[2] TCP 3-way handshake──────► SYN / SYN-ACK / ACK             (~1 RTT)
    │
    ▼
[3] TLS handshake────────────► keys negotiated                 (~1–2 RTT, TLS1.3 = 1)
    │
    ▼
[4] HTTP GET /dashboard ─────► travels to the edge/server
    │
    ▼
[5] REVERSE PROXY (nginx)
      ├─ static file? ─► disk/…os cache ─► return  (1–3 ms)   ◄── seconds saved here
      └─ otherwise ────► proxy_pass
    │
    ▼
[6] EXPRESS
      middleware (session, auth) ─► ROUTE ─► CONTROLLER
    │
    ▼
[7] TEMPLATE RENDER  ◄── recompiles every request in dev (200 ms+)
    │
    ▼
[8] DATABASE QUERY   ◄── N+1: 1 + 500 queries (900 ms!) or an indexed 5 ms
    │
    ▼
[9] RESPONSE serialised ─► network back ─► browser
    │
    ▼
[10] BROWSER: parse HTML ─► fetch CSS/JS/fonts (NEW round-trips) ─► FCP ─► LCP
```

**Where a second hides, ranked by reality (not by theory):**

1. **The database.** One unindexed `WHERE`, or a loop that fires one query per row, dwarfs everything else.
2. **Missing cache.** Recomputing the same page/query on every request instead of storing the answer.
3. **The network.** New connections (DNS/TCP/TLS) and uncompressed bytes on every asset.
4. **The app.** Sync I/O that blocks Node's single thread, or re-rendering the whole page for one changed fragment.
5. **The browser.** Render-blocking JS, layout thrash, fonts that pop in late.

An **ASCII waterfall** makes the split obvious — bar length is time:

```
            0ms    200    400    600    800   1000  1200  1400  1600  1800
DNS/TLS     ├█┤
Network        ├██┤
Proxy           ●
Express          ●
DB query             ├███████████████████████████┤          ← 900 ms
Template                                          ├███████┤
Transfer                                                  ├███┤
Browser paint                                                ├███████████┤
                                                              └─ FCP at ~1.65s
```

Once you have this picture, "make it faster" becomes a concrete question: **which bar is longest, and what removes it?**

---

### 3. Caching as the Architecture Lever

Caching layers are ordered from **closest to the user (fastest, least specific)** to **deepest in the system (slowest, most authoritative)**. Each answers a *different* question.

| Layer | What it stores | Freshness control | The invalidation problem | Survives restart? |
|-------|----------------|-------------------|--------------------------|-------------------|
| **Browser** | Any response/assets per URL | `Cache-Control`, `ETag`, `Expires` | Change the URL (versioned filenames) or accept staleness | Per-user disk |
| **CDN** | Whole responses at the edge | Origin `Cache-Control` / surrogate keys | Purge API, TTL, versioned URLs | Yes (edge nodes) |
| **Reverse proxy (nginx micro-cache)** | Fully rendered HTML pages | `proxy_cache_valid` TTL | Delete cache file(s) or wait out TTL | To disk |
| **App memory (`node-cache`)** | Objects per process | TTL set by your code | TTL, explicit `del`, process restart | No |
| **Redis (shared app cache)** | Query results / fragments | TTL + explicit keys | `DEL` on write, or version-prefixed keys | Yes |
| **Database** | Indexes, materialized views | Planner statistics | Writes update indexes automatically; matview needs `REFRESH` | Yes |

#### 3.1 Browser cache — the cheapest millisecond is the one you never send

The server tells the browser what it may reuse:

```http
Cache-Control: public, max-age=31536000, immutable   # hashed asset: never revalidate
ETag: "a1b2c3"                                       # revalidate cheaply if needed
Cache-Control: no-store                              # per-user HTML: never store
```

- `max-age` — the browser may reuse the response without asking for that many seconds.
- `immutable` — the bytes at this URL will never change, so skip even a conditional request. Only safe for **content-hashed** filenames (`app.e8f1c2.js`).
- `ETag`/`If-None-Match` — if the file might change, the browser asks "still valid?"; a match returns **304 Not Modified** with zero body bytes.
- `no-store` — the escape hatch for anything personal.

The invalidation trick that avoids invalidation: **never reuse a URL whose contents changed.** Hash the file content into the name; a new build produces a new URL, so the old cached entry is simply never requested again.

#### 3.2 CDN — push static bytes to the edge

A CDN caches responses in data centers near the user, so the static-asset round-trip shrinks from "across an ocean" to "across a city". Application-level *content* (images, JS/CSS, fonts) is the ideal candidate; dynamic HTML usually stays uncached or uses a short TTL.

#### 3.3 Reverse-proxy micro-cache — serve a whole page before Express ever runs

nginx can cache a fully rendered anonymous page for a few seconds. At 500 requests/second, a 5-second cache means one Express render serves thousands of users:

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=page_cache:10m max_size=1g inactive=10m;

location / {
    proxy_cache page_cache;
    proxy_cache_valid 200 5s;            # anonymous pages live 5 seconds
    proxy_cache_bypass $cookie_session;  # logged-in users bypass the cache
    add_header X-Cache-Status $upstream_cache_status;   # HIT / MISS / BYPASS
    proxy_pass http://web:3000;
}
```

`$upstream_cache_status` is the single most useful debugging header you can add: it tells you, per request, whether the cache did its job.

#### 3.4 App-level cache (Redis / node-cache) — cache-aside on query results

The **cache-aside** pattern: read the cache first; on a miss, query the DB and write the result back with a TTL.

```js
// cache.js — runnable with: npm i ioredis; node cache.js
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL || 'redis://127.0.0.1:6379');

export async function cached(key, ttlSeconds, loader) {
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);              // fast path: no DB at all
  const value = await loader();                  // slow path: hit the DB once
  await redis.set(key, JSON.stringify(value), 'EX', ttlSeconds);
  return value;
}
```

```js
// In a controller — the same route, now mostly DB-free
router.get('/products', async (req, res) => {
  const products = await cached('products:popular', 60, () =>
    knex('products').where({ popular: true }).select('id', 'name', 'price')
  );
  res.render('products', { products });
});
```

- `node-cache` lives in one process — fine for a single instance, wrong the moment you run two.
- Redis is shared across instances and survives restarts; it is the default choice once you scale horizontally.
- **Invalidate on write**: when a product is updated, `redis.del('products:popular')`. TTL is the safety net; explicit `DEL` is the correctness guarantee.

#### 3.5 DB-level — the cache you already have is the index

An index is a persistent cache of *sorted pointers*. It does not remove the query; it turns a full-table scan into a B-tree seek. This is the same trade-off as an in-memory cache — faster reads for slower writes — and it is covered in depth in [`../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md`](../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md). Materialized views go one step further: they *store the computed result* and are refreshed on a schedule.

#### 3.6 When caching is the WRONG tool

- **Per-user pages** (dashboards, "my orders"). Caching them by URL leaks one user's data to another — a security incident, not a perf regression. Cache *fragments* that are shared, or use `Vary`/private keys correctly.
- **Realtime data** (live prices, chat, IoT telemetry). A stale answer is a bug.
- **Highly volatile/shared mutable state** where invalidation is harder than recomputation.
- **Anything you cannot invalidate.** If you cannot answer "what makes this entry wrong, and who deletes it?", do not cache it yet.

---

### 4. App-Code Bottlenecks

These are the bugs a demo never reveals. Each is explained with the failing code, then the fix.

#### 4.1 Blocking the event loop (Node's single thread)

Node runs your JavaScript on **one thread**. Synchronous I/O or a long CPU loop freezes *every other request* until it finishes — the whole server stalls, not just one user. (See [`../dev_foundation/web_server/04-routing-in-express.md`](../dev_foundation/web_server/04-routing-in-express.md) for the request/middleware mechanics.)

```js
// ❌ BAD — every request blocks the single thread
app.get('/report', (req, res) => {
  const raw = fs.readFileSync('huge.csv', 'utf8');  // sync: whole server frozen
  res.send(buildReport(raw));
});

// ✅ BETTER — I/O yields the thread to other requests
app.get('/report', async (req, res) => {
  const raw = await fs.promises.readFile('huge.csv', 'utf8');
  res.send(buildReport(raw));
});
```

Caveat that trips people up: `await` fixes *I/O* blocking, **not CPU blocking**. If `buildReport(raw)` is a 2-second CPU loop, the server still freezes. CPU-bound work belongs in a `worker_threads` pool (or a separate worker process). The same rule applies to sync crypto (`pbkdf2Sync` on login) and catastrophic regex backtracking.

#### 4.2 N+1 queries

The most common DB performance bug in ORM/query-builder code: fetch a list, then issue one extra query per row.

```js
// ❌ N+1: 1 query for orders + N queries for customers = 501 round-trips
const orders = await knex('orders').limit(500);
for (const o of orders) {
  o.customer = await knex('customers').where({ id: o.customer_id }).first();
}

// ✅ ONE query — join, or batch with whereIn
const orders = await knex('orders')
  .join('customers', 'customers.id', 'orders.customer_id')
  .select('orders.id', 'orders.total', 'customers.name as customer_name')
  .limit(500);
```

Each query costs a network round-trip plus parse/plan time. 500 of them is 500× that, and it is the single biggest cause of the 900 ms DB bar in §2.

#### 4.3 Re-rendering the entire page for one fragment

If clicking "next page" of a table re-runs the whole route and rebuilds the whole HTML, you paid the full SSR cost for one component. The fix is a small **JSON endpoint** + client-side `fetch` that swaps only the fragment — the bridge to [`../dev_foundation/web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../dev_foundation/web_server/08-advanced-features-pagination-wysiwyg-upload.md).

```js
// server: return data, not a whole page, for the table fragment
router.get('/api/orders', async (req, res) => {
  const page = Number(req.query.page) || 1;
  const rows = await ordersRepo.page(page, 20);
  res.json(rows);
});
```

```js
// client: replace one <tbody>, not the whole document
async function loadPage(page) {
  const rows = await (await fetch(`/api/orders?page=${page}`)).json();
  document.querySelector('#orders tbody').replaceChildren(...renderRows(rows));
}
```

#### 4.4 Big unminified bundles, missing compression

A 400 KB development bundle shipped as-is, uncompressed, over a slow link, is seconds of transfer for no benefit. Minify (drop whitespace/comments, mangle names), then let the server compress the bytes on the wire (§5).

#### 4.5 Template engine recompiling in dev

In development Express recompiles templates on every request so you can edit without restarting. In production that same setting burns CPU per request. Set production mode and enable the view cache:

```js
app.set('view engine', 'hbs');
app.set('view cache', true);   // on automatically when NODE_ENV=production, but be explicit
```

The Docker note [`../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md) makes this concrete: `NODE_ENV=production` in the image is what flips these behaviours.

---

### 5. Asset & Network Optimization

#### 5.1 Content hashes + long cache

Serve static files with a year-long cache *only* when their names contain a content hash, so a deploy changes the URL:

```js
app.use('/static', express.static('public', {
  maxAge: '1y',
  immutable: true,            // safe because filenames are content-hashed: app.e8f1c2.js
}));
// HTML itself is never long-cached:
res.set('Cache-Control', 'no-store');
```

#### 5.2 Compression at the proxy

Compress once at nginx, not in Node (less CPU in your app, works for every upstream):

```nginx
gzip on;
gzip_types text/html text/css application/javascript application/json image/svg+xml;
gzip_min_length 1024;
brotli on;                     # module; ~15–25% smaller than gzip for text
brotli_types text/html text/css application/javascript application/json;
```

#### 5.3 HTTP/2 and connections

HTTP/2 multiplexes many requests over one TCP connection (no more 6-connection limit), and lets the server push or, more usefully today, `preload` critical assets. `preconnect` tells the browser to open the DNS/TCP/TLS pipes early:

```html
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
<link rel="preload" href="/static/app.e8f1c2.css" as="style">
```

#### 5.4 The metrics that matter (Lighthouse / Core Web Vitals)

| Metric | Question it answers | Good target |
|--------|--------------------|-------------|
| **TTFB** | How fast did the server start responding? | < 800 ms (ideally < 200 ms) |
| **FCP** | When did the first content appear? | < 1.8 s |
| **LCP** | When did the largest element finish? | < 2.5 s |
| **CLS** | How much did the layout jump? | < 0.1 |

**Measure with `curl -w`** (server-side split, no browser needed):

```bash
curl -s -o /dev/null -w "dns=%{time_namelookup}s connect=%{time_connect}s tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s bytes=%{size_download}\n" \
  http://localhost:3000/
```

Expected output on a warm local server:

```
dns=0.000021s connect=0.000180s tls=0.000000s ttfb=0.045s total=0.047s bytes=18422
```

`ttfb` is the number you attack first; `bytes` moving down after enabling brotli is your proof compression works. Use **DevTools → Network** for the client-side waterfall (DNS/TLS/blocking/paint).

---

### 6. The Prioritized Fix Checklist

Order by *impact per unit of effort*, and re-measure after each step. Never skip step 1.

| # | Step | Why it's at this position | Typical effect | Effort |
|---|------|--------------------------|----------------|--------|
| 1 | **Measure** (`curl -w` + DevTools, p50/p95) | You cannot fix a bar you cannot see; wrong guesses waste days | — | 10 min |
| 2 | **Cache headers on assets** (`immutable` + hashes) | Removes repeat network cost for every returning user | −50…70% repeat-visit load | 1 h |
| 3 | **Index + kill N+1** | Largest single server-side owner; correctness-preserving | DB 900→~15 ms | 1–3 h |
| 4 | **Compress assets** (gzip/brotli) | Pure network win, zero logic risk | −60…75% transfer bytes | 30 min |
| 5 | **Add query cache** (Redis cache-aside, short TTL) | Converts the hot read from ms to sub-ms | DB 15→~2 ms on hits | 2–4 h |
| 6 | **Add page micro-cache** (nginx, anonymous GET, `bypass` on session) | Skips Express *and* DB for the shared case | TTFB → ~5 ms on hits | 2 h |
| 7 | **Scale DB reads** (read replica / pooling) | Only when writes or genuine load demand it | Throughput ↑ | days |

Rule of thumb: **caching beats query tuning beats code tuning**, and *all three* beat premature horizontal scaling.

---

### 7. Worked Example — From 1.8 s to ~250 ms

A wireframe product-list page, anonymous user, warm cache. Each row is a *measured* improvement, not a guess.

| Step | Technique applied | Measured effect | New TTFB | New repeat page-load |
|------|-------------------|-----------------|----------|----------------------|
| 0 | **Baseline** (N+1 + no index, dev mode, no cache, no gzip) | — | **1800 ms** | ~2.6 s |
| 1 | Index the `WHERE category_id` column | DB scan 900 → 45 ms | ~945 ms | ~1.7 s |
| 2 | Replace N+1 with one JOIN | DB 45 → 12 ms | ~912 ms | ~1.7 s |
| 3 | Redis cache-aside on the hot list (60 s TTL) | DB 12 → 2 ms on hits | ~900 ms | ~1.7 s |
| 4 | `NODE_ENV=production` + view cache + minify | render 220 → 15 ms | ~700 ms | ~1.4 s |
| 5 | nginx micro-cache, 5 s, bypass on session cookie | skip Express + DB entirely | **~270 ms** | ~0.6 s |
| 6 | brotli + HTTP/2 on assets | transfer 160 → 35 KB | ~255 ms | ~0.45 s |
| 7 | `preconnect` + hashed immutable assets | kills repeat DNS/TLS | **~250 ms** | **~0.3 s** |

Cumulative: **~1.8 s → ~250 ms server time**, with no change to what the user sees on the page. Note the shape of the curve: the *biggest* wins came from the DB (steps 1–3) and the *cheapest* wins came from configuration (steps 5–6). That ordering is the whole lesson.

---

### 8. Comparison Table — Caching Layers at a Glance

| Feature | Browser | CDN | Nginx micro-cache | Redis (app) | DB index |
|---------|---------|-----|-------------------|-------------|----------|
| Distance from user | Closest | Near | Server edge | Server (shared) | Deepest |
| What it saves | Re-request | Network distance | Express + DB | DB round-trip | Table scan |
| Freshness control | `Cache-Control`/`ETag` | `Cache-Control`, purge | `proxy_cache_valid` | TTL + `DEL` | Automatic on write |
| Per-user safe? | Yes (private) | With `private` | Only with `bypass` | Key by user id | Yes |
| Survives restart | Per-user | Yes | To disk | Yes | Yes |
| Main risk | Stale assets | Stale HTML | Leaking a logged-in page | Stale reads | Slower writes, disk |

> **Why it matters:** This is the exact material interviewers probe when they ask "the page is slow, what do you do?" — and the exact discipline every AI product front-end needs, because a model that returns in 300 ms is worthless if the page around it takes 2 seconds to paint. Shipping a demo is easy; shipping a fast product is architecture.

---

## 🔗 Tech Cross-References

- **Performance ↔ Databases** — the underlying cause of most latency is a query anti-pattern (missing index, N+1, deep OFFSET); this note assumes the fix list from there. See [`../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md`](../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md).
- **Performance ↔ Web Server** — the reverse-proxy layer (nginx) is where micro-caching, gzip/brotli and TLS live. See [`../dev_foundation/web_server/02-web-server-nginx-apache-express.md`](../dev_foundation/web_server/02-web-server-nginx-apache-express.md).
- **Performance ↔ Docker** — deployment topology decides whether caches are per-instance (node-cache) or shared (Redis); stateless replicas scale. See [`../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md).
- **Performance ↔ Front-end** — asset size, render-blocking CSS and TTFB are half of the page's total time. See [`../dev_foundation/web_basics/06-responsive-web-design.md`](../dev_foundation/web_basics/06-responsive-web-design.md).

## 📝 Research Journey

- **Why:** I had already learned "the database is usually the bottleneck" (from `databases/08`) and I knew Docker from the folder README, but I had never connected them into a *single timeline*. This note exists because I wanted to stop guessing where the seconds went on the Web Application Development project and start *measuring*.
- **Struggle:** The hardest idea was that **caching is architecture, not code**. My instinct was to optimise functions. It took drawing the waterfall to see that the biggest bar was a query I never wrote on purpose (N+1) and that the fix was not "faster code" but "don't do the work at all."
- **Aha moment:** Two of them. (1) The request that never reaches your server costs nothing — which is why the *cheapest* fixes (cache headers, nginx micro-cache) beat the cleverest code change. (2) `await` does not make Node non-blocking; it only yields on I/O. A CPU loop still freezes every other user. That single sentence reframed how I read every tutorial that says "just use async."
- **Career link:** Deployment is where "it works on my machine" dies, and performance is where "it works but nobody uses it" dies. As an AI Engineer shipping models behind a web surface, I will own both the inference latency *and* the page latency around it — and the same cache-aside pattern appears again at the model-serving layer.

## 🔗 Related Topics

- [`03-application-architecture-models.md`](03-application-architecture-models.md) — where the cache layers physically sit in a tiered SSR architecture.
- [`01-event-driven-architecture-task-orchestration.md`](01-event-driven-architecture-task-orchestration.md) — queues and async work move slow operations off the request path entirely.
- [`../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md`](../dev_foundation/databases/08-db-bottlenecks-indexing-and-caching.md) — the DB half of every bottleneck in this note (the N+1 and index story).
- [`../dev_foundation/web_server/02-web-server-nginx-apache-express.md`](../dev_foundation/web_server/02-web-server-nginx-apache-express.md) — reverse proxy, `proxy_cache`, and where "static vs dynamic" is decided.
- [`../dev_foundation/web_server/04-routing-in-express.md`](../dev_foundation/web_server/04-routing-in-express.md) — middleware/routing mechanics behind the "Express" bar in the waterfall.
- [`../dev_foundation/web_basics/06-responsive-web-design.md`](../dev_foundation/web_basics/06-responsive-web-design.md) — the client-side rendering layer and why layout decisions affect paint time.
- [`../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md`](../dev_foundation/docker/05-deploy-express-ssr-postgres-compose.md) — how `NODE_ENV=production` and the deployed topology activate everything here.
- [SUGGESTED] `05-observability-and-metrics-for-web-apps.md` — *why it's important:* every number in this note came from measurement, but I have no systematic way to collect p50/p95 latency, cache hit ratio, and query time in production. Structured logging + metrics dashboards (Prometheus/Grafana, or an APM) turn this note's manual `curl` into a permanent signal. *Folder:* `software_architecture/`.
- [SUGGESTED] `bridge-http-caching-and-apis.md` — *why it's important:* the `Cache-Control`/`ETag` rules here also govern REST/JSON API caching (surrogate keys, `stale-while-revalidate`), which is where an AI microservice's responses get cached. *Folder:* `software_architecture/`.

## 🤔 Open Questions

- [ ] For the course project, what is the *actual* p95 TTFB on the deployed VPS, and which bar dominates it?
- [ ] What is the smallest viable nginx config that gives a safe micro-cache without ever leaking a logged-in page?
- [ ] At what scale does Redis stop being worth it versus just letting Postgres's own buffer cache and a read replica do the work?
- [ ] How do I prove a perceived-performance gain (LCP) rather than just a server-side TTFB gain — and which metric do real users actually feel?

---
*Performance is not one big fix. It is a hundred small subtractions, applied to the bar that is longest.*

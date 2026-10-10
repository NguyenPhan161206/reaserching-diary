# Web Server Mental Model — Nginx · Apache · Express

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#WebServer`, `#HTTP`, `#Nginx`, `#Apache`, `#Express`, `#NodeJS`, `#ReverseProxy`, `#DevTools`
**References:** Nginx docs — `ngx_http_proxy_module`, `ngx_http_core_module`; Apache HTTP Server docs — "Virtual Hosts", `.htaccess`; Express.js 4 guide — "Routing", "Using middleware", `express.static`; Node.js docs — `http` module; MDN "HTTP" & "HTTP headers"
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`../../linux/02-navigation-and-core-cli.md`](../linux/02-navigation-and-core-cli.md) (shell, `curl`, ports), [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md) (JS syntax for Express)

---

## 📌 Key Takeaways (TL;DR)
- A **web server is a process bound to a socket** (`IP:PORT`) that *reads* an HTTP request and *writes* an HTTP response. Nothing renders until a socket accepts a connection — so diagnosis always starts with **"which process, which port"**.
- "Web server" usually means up to **three layers**: **TLS termination / static delivery / reverse proxy** (Nginx or Apache) in *front*, and the **application server** that runs your logic (Express) *behind* it.
- **Express is not a replacement for Nginx** — it is the application layer. Nginx handles port 80/443, static files, TLS, and gzip; Express handles routes and data.
- The debugger's view of HTTP is `curl -i`: it prints the **exact bytes** the server returns (status line, headers, body). If you cannot explain what `curl` shows, you do not yet understand the response.
- `proxy_pass` errors and "port already in use" errors are 90% of first-day server bugs.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

Before any command: **a web server is a process listening on a socket.** HTTP is just text sent across that socket. Everything else — routing, templates, databases — happens *after* the socket accepts the connection.

```
        A socket is the entry point
        ───────────────────────────
   TCP connect                accept()                handle
browser ──────────►  IP:PORT  ──────►  server process ─────► read request
 (client)           socket             (Node/Nginx)          write response
```

**The three layers of "a web server"** — the phrase hides all of this:

```
   ┌───────────────────────────────────────────────────────────┐
   │  Layer 3: application logic                                │
   │  Express (Node) — routes, controllers, templates, data     │
   └────────────────────────────▲──────────────────────────────┘
                                │ proxy_pass http://127.0.0.1:3000
   ┌────────────────────────────┴──────────────────────────────┐
   │  Layer 2: reverse proxy / static / TLS                     │
   │  Nginx or Apache — :80/:443, serve /assets, gzip, TLS      │
   └────────────────────────────▲──────────────────────────────┘
                                │ HTTP
   ┌────────────────────────────┴──────────────────────────────┐
   │  Layer 1: the network + OS                                 │
   │  TCP/IP, the listening socket, file descriptors, ports     │
   └───────────────────────────────────────────────────────────┘
```

**Why this mental model matters:** when something "doesn't work", the layers localise the fault. A `502 Bad Gateway` is *the proxy* telling you it reached the network but the **application behind it is down**. `EADDRINUSE` is *the OS* telling you the **port is already taken**. `404` is *your app* telling you the **route does not exist**. Each error names a different layer — and the layer tells you which process to inspect.

**Diagnosis always starts with two questions:** *which process?* and *which port?* (`ps`, `ss -ltnp`, `lsof -i :3000`). See [`../../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) for the process/port tooling.

### 2. HTTP on the Wire

An HTTP request is plain text with a strict shape:

```
POST /posts?draft=1 HTTP/1.1        ← request line: METHOD  PATH  VERSION
Host: localhost:3000                ← headers: key: value
Content-Type: application/json
Content-Length: 27
                                    ← blank line separates headers from body
{"title":"Hello","body":"World"}    ← optional body
```

The response mirrors it:

```
HTTP/1.1 201 Created                ← status line: VERSION  CODE  REASON
Content-Type: application/json      ← headers
Content-Length: 31
                                    ← blank line
{"id":42,"title":"Hello"}           ← body
```

**Status-code families (memorise the class, not every number):**

| Range | Meaning | Common in this course |
|---|---|---|
| **2xx** | Success | `200 OK`, `201 Created` |
| **3xx** | Redirection | `301`/`302` (login redirect) |
| **4xx** | Client error | `400`, `401`, `403`, `404` |
| **5xx** | Server error | `500` (your bug), `502` (proxy: app down) |

**`curl` is the debugging view into all of this.** `curl -i` prints the status line + headers + body exactly as the server wrote them — no browser magic, no DevTools filtering:

```bash
curl -i http://localhost:3000/posts
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=utf-8
# Content-Length: 53
#
# <html><body><h1>Posts</h1></body></html>
```

**Why it matters:** a browser renders a response and hides the raw exchange. `curl` shows the truth. When you are unsure whether *your server* is wrong or *your front end* is wrong, `curl` the endpoint directly — if `curl` looks right, the server is fine and the bug is client-side.

### 3. From Node's `http` Module to Express

**The minimal server — no framework.** Node ships an `http` module; a server is a function that runs on every request:

```js
// minimal.js — a real, runnable server
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello from a raw Node server\n");
});

server.listen(3000, () => console.log("listening on http://localhost:3000"));
```

Run it, then in another terminal: `curl -i http://localhost:3000`. You get `200` and the text. That is a web server — everything else is convenience.

**Express = routing + middleware on top of `http`.** It keeps Node's server underneath but adds structure:

- **Routing** — map `METHOD + PATH` to a handler: `app.get`, `app.post`, `app.use`.
- **Middleware** — functions that run *in order* before the handler, each able to inspect/modify the request, response, or stop the chain (`next(err)`).

```js
// express-mini.js — runnable after: npm i express
const express = require("express");
const app = express();

app.use(express.json());            // middleware: parse JSON bodies → req.body
app.use(express.static("public"));  // middleware: serve files from ./public

app.get("/", (req, res) => res.send("Home page"));       // res.send → text/HTML
app.get("/posts/:id", (req, res) => {                    // route param
  res.json({ id: req.params.id, q: req.query });         // res.json → JSON
});
app.post("/posts", (req, res) => {
  res.status(201).json({ created: true, body: req.body });// status + JSON
});
app.get("/old", (req, res) => res.redirect("/"));         // 302 redirect

app.listen(3000, () => console.log("Express on :3000"));
```

**The `req` / `res` surface you will use constantly:**

| Object | Property/method | What it gives you |
|---|---|---|
| `req` | `req.params` | path variables (`/posts/:id` → `{ id }`) |
| `req` | `req.query` | query string (`?q=a&page=2`) |
| `req` | `req.body` | parsed POST body (**needs `express.json()`**) |
| `res` | `res.send` / `res.json` | send text/HTML or JSON |
| `res` | `res.render` | render a template → full HTML (the SSR bridge) |
| `res` | `res.redirect` | send a `3xx` to another URL |
| `res` | `res.status` | set the status code |

**`app.listen(PORT)` binds the socket** — this is the line that makes the process a *server* rather than a script that exits. During development, **`nodemon`** watches files and restarts the server on save, so you do not have to Ctrl-C and rerun.

### 4. Nginx — the Front Layer

Nginx is a **high-performance, event-driven** server used as a **static file server**, **reverse proxy**, **TLS terminator**, and **rate limiter**. In this course's architecture it sits in *front* of Express.

**Config anatomy of a reverse proxy** (`/etc/nginx/sites-available/app`):

```nginx
server {
    listen 80;                              # Nginx owns port 80
    server_name example.com;

    location /assets/ {                     # static files served BY NGINX
        root /var/www/app/public;
    }

    location / {                            # everything else → Express
        proxy_pass http://127.0.0.1:3000;   # forward to the Node app
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Why put Nginx in front of Node?** Node can listen on 3000 itself — so why the extra layer?

| Reason | Explanation |
|---|---|
| **Port 80/443 access** | A non-root Node process cannot bind privileged ports; Nginx can, and proxies inward |
| **Static offload** | Nginx serves images/CSS/JS far more efficiently than your Node app, freeing it for logic |
| **Single entry point** | One public address for many backends; TLS and routing in one place |
| **gzip / caching / rate limiting** | Compression and abuse protection without writing app code |
| **TLS termination** | HTTPS is handled at the edge; the app speaks plain HTTP internally |

**Reload vs restart.** Nginx reloads config **gracefully** — existing connections drain, new ones use the new config, no dropped requests:

```bash
sudo nginx -t                    # 1. TEST the config first (always!)
sudo systemctl reload nginx      # 2. graceful reload
# equivalent: sudo nginx -s reload
```

`nginx -t` parses the config and reports syntax errors *before* you touch the running server — skipping it is how you take a site down with a typo.

### 5. Apache — the Alternative Front Layer (Brief)

Apache HTTP Server is the other classic front layer. Its model differs in ways that change how you write config:

- **Virtual hosts** — one server, many sites, separated by `ServerName` and `DocumentRoot`:
  ```apache
  <VirtualHost *:80>
      ServerName example.com
      DocumentRoot /var/www/app/public
      ErrorLog  ${APACHE_LOG_DIR}/error.log
  </VirtualHost>
  ```
- **`.htaccess`** — per-directory override files that let config change *without* touching the main server config or restarting (convenient on shared hosting, a performance cost when `AllowOverride` is on).
- **Process-per-request** — Apache traditionally spawns a process/thread per connection (prefork/worker MPMs), which is simple but scales differently than Nginx's event loop.

**Nginx vs Apache — which to pick when:**

| Aspect | Nginx | Apache |
|---|---|---|
| Concurrency model | Event-driven, async (few workers, many connections) | Process/thread per request (MPM-dependent) |
| Static file speed | Excellent | Good |
| Config location | Central `server` blocks | Central config **+ `.htaccess`** per directory |
| Config reload | Graceful reload, no restart | `reload` also supported |
| Module ecosystem | Smaller, compiled in | Large, historic, dynamic modules (`.so`) |
| `.htaccess` | ❌ none | ✅ per-directory overrides |
| Best for | High-concurrency reverse proxy / static edge | Legacy apps, shared hosting, `.htaccess` workflows |

**Rule of thumb:** for a modern Node/Express deployment, **Nginx** is the default front layer. Reach for **Apache** when you inherit a `.htaccess`-based app, need a specific module, or work on classic LAMP/shared hosting.

### 6. Full Topology for the Course

Putting the layers together — the architecture this course builds toward:

```
 Browser
   │  HTTPS :443
   ▼
┌──────────────────────────────┐
│  Nginx  (reverse proxy / TLS) │
│  • /assets/* → static files   │
│  • /          → proxy_pass    │
└──────────────┬───────────────┘
               │  http://127.0.0.1:3000
               ▼
┌──────────────────────────────┐
│  Express app  (Node)          │
│  • routes / middleware / MVC  │
│  • res.render(...) → HTML     │
└──────────────┬───────────────┘
               │  SQL (Knex)
               ▼
┌──────────────────────────────┐
│  PostgreSQL                   │
└──────────────────────────────┘
```

- **Static assets** live at Nginx (fast, cached).
- **Dynamic routes** (`/`) proxy to Express.
- **Data** comes from PostgreSQL via the app's data layer.
- In production this whole stack is often packaged with Docker Compose — see [`../../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md).

---

## 🧾 Command / Code Cheat Sheet

| Command / Code | What it does | When to use | Example / Output |
|---|---|---|---|
| `curl -i URL` | Show status line + headers + body | The first debugging step for any endpoint | `HTTP/1.1 200 OK …` |
| `curl -X POST -H "Content-Type: application/json" -d '{...}' URL` | Send a JSON POST | Testing a write endpoint without a browser | `201 Created` |
| `http.createServer((req,res)=>…)` | Raw Node HTTP server | Understanding what a server *is*, before Express | serves text for any path |
| `app.get(path, fn)` / `app.post(path, fn)` | Register routes | Map URL + method to handlers | `GET /posts` → handler |
| `app.use(mw)` | Mount middleware | JSON parsing, static files, logging, auth — order matters | runs on every request |
| `app.listen(PORT, cb)` | Bind the socket and start serving | The line that makes it a server | `listening on :3000` |
| `nodemon server.js` | Auto-restart on file change | Development loop | restarts on save |
| `nginx -t` | Test config syntax | **Always** before reloading | `syntax is ok` / `test is successful` |
| `systemctl reload nginx` (or `nginx -s reload`) | Graceful config reload | After editing a config file | no dropped connections |
| `systemctl status nginx` | Is Nginx running, and its logs? | "502 / connection refused" triage | `active (running)` |
| `proxy_pass http://127.0.0.1:3000;` | Forward requests to the app | Reverse-proxy config | browser hits :80, Node answers |
| `ss -ltnp` / `lsof -i :3000` | Which process holds which port | `EADDRINUSE`, "wrong app answered" | `node … :3000` |
| `kill <pid>` | Stop a process by id | Free a port / stop a runaway server | process terminates |

---

## 🧪 Practical Lab

> Two labs, fully runnable, each with the expected output to verify against.

**Lab 1 — a minimal Express server, verified with `curl -i`**

```bash
mkdir /tmp/ws-lab && cd /tmp/ws-lab
npm init -y
npm i express

cat > server.js <<'EOF'
const express = require("express");
const app = express();
app.use(express.json());

app.get("/", (req, res) => res.send("Home page"));
app.get("/posts/:id", (req, res) => res.json({ id: req.params.id }));
app.post("/posts", (req, res) => res.status(201).json({ created: true, body: req.body }));

app.listen(3000, () => console.log("listening on http://localhost:3000"));
EOF

PORT=3000 node server.js
# Expected (server terminal): listening on http://localhost:3000
```

In a **second terminal**:

```bash
curl -i http://localhost:3000/
# HTTP/1.1 200 OK
# X-Powered-By: Express              ← the header you should hide in production
# Content-Type: text/html; charset=utf-8
# Content-Length: 9
#
# Home page

curl -i http://localhost:3000/posts/42
# HTTP/1.1 200 OK
# Content-Type: application/json; charset=utf-8
#
# {"id":"42"}

curl -i -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Hello"}'
# HTTP/1.1 201 Created
# Content-Type: application/json; charset=utf-8
#
# {"created":true,"body":{"title":"Hello"}}
```

*What to notice:* `res.send` produced `text/html`, `res.json` produced `application/json`, and `req.body` was populated **only because `express.json()` was mounted**. Remove that middleware and the POST body comes back `{}` — that is the classic missing-parser bug.

**Lab 2 — Nginx in front of Express**

```bash
# server.js from Lab 1 still running on :3000.
# Create an Nginx site config (adjust paths for your install):
sudo tee /etc/nginx/sites-available/ws-lab >/dev/null <<'EOF'
server {
    listen 80;
    server_name _;

    location /assets/ {
        root /tmp/ws-lab/public;      # Nginx serves static files itself
    }
    location / {
        proxy_pass http://127.0.0.1:3000;   # the app behind the proxy
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF

sudo ln -sf /etc/nginx/sites-available/ws-lab /etc/nginx/sites-enabled/ws-lab
sudo nginx -t                 # EXPECTED: "syntax is ok" + "test is successful"
sudo systemctl reload nginx   # graceful pickup of the new config

curl -i http://localhost/     # NOTE: port 80, not 3000
# HTTP/1.1 200 OK
# X-Powered-By: Express
# Content-Type: text/html; charset=utf-8
#
# Home page
```

*What to notice:* the browser talks to **port 80** (Nginx), but the content is produced by **Express on 3000** — the `/` location proxied it. If you stop the Node process and `curl` again, you get **`502 Bad Gateway`**: Nginx is alive, the app behind it is dead. That error *is* the layer diagnosis from §1.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---|---|---|---|
| Port already in use | `Error: listen EADDRINUSE :::3000` | Another process (an old server you forgot) owns the port | Find it: `lsof -i :3000` / `ss -ltnp`, then `kill <pid>` |
| Forgetting `express.json()` | POST body arrives as `{}` | No middleware parsed the JSON body | `app.use(express.json())` **before** the routes |
| Middleware order wrong | Auth/parser "not running", or runs after the handler | Middleware runs in registration order; `app.use` must precede the routes it protects | Register parsing/static/auth **above** the routes |
| `proxy_pass` missing/extra trailing slash | Paths shift (`/api/x` vs `/x`) | Nginx rewrites the URI depending on whether `proxy_pass` ends in `/` | Match the slash deliberately; test with `curl -i` |
| Editing `nginx.conf` without reloading | Changes have no effect | Nginx keeps the old config in memory until reload | `nginx -t && systemctl reload nginx` |
| Leaking server headers | `X-Powered-By: Express`, `Server: nginx` visible | Defaults advertise the stack — a hint for attackers | `app.disable("x-powered-by")`; `server_tokens off;` in Nginx |
| CORS when front end ≠ API host | Browser console: "blocked by CORS policy" | Browsers block cross-origin requests without consent headers | Serve same-origin behind the proxy, **or** set `Access-Control-Allow-Origin` deliberately |
| Binding to `127.0.0.1` vs `0.0.0.0` | Works locally, unreachable from another machine/container | `127.0.0.1` accepts only local connections | `app.listen(3000, "0.0.0.0")` when it must be reachable |
| `502 Bad Gateway` | Nginx returns 502 | Proxy is up; the app behind it is down or not on that port | `systemctl status` the app; `ss -ltnp` the port; check the app log |
| Not testing config before reload | Whole site goes down on a typo | A broken config replaces a working one | **Always** `sudo nginx -t` first |
| Hard-coding the port | Deploy fails on another machine | Port fixed in code instead of from the environment | `const PORT = process.env.PORT || 3000` |

---

## 🔗 Tech Cross-References
- **Linux (processes/ports)** — a server is a process; `lsof`/`ss`/`kill` are how you find and stop it. Ports and PIDs are the first diagnostic surface. See [`../../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md).
- **JavaScript (client vs server)** — Express is the *server* runtime of the same language you write in the browser; `require` vs `import`, `process` vs `window`. See [`../web_basics/04-javascript-language-core.md`](../web_basics/04-javascript-language-core.md).
- **Docker / deployment** — this exact Nginx→Express→Postgres topology is packaged and run with Compose; the container names replace the ports you used here. See [`../../docker/05-deploy-express-ssr-postgres-compose.md`](../docker/05-deploy-express-ssr-postgres-compose.md).
- **Security** — headers, reverse-proxy trust (`X-Real-IP`), and injection/XSS/session concerns all begin at this request boundary. See [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).

## 🧪 Practice Tasks
> No solutions provided. Do them, then ask the AI to grade your evidence (the `curl -i` output and config).

- [ ] **Task 1 — Port detective.** Start two Express servers that both call `app.listen(3000)`. Capture the exact error text, then write the exact commands you used to identify and kill the first process. Success = you can explain *which layer* reported the error.
- [ ] **Task 2 — JSON parser race.** Build a POST `/echo` route that returns `req.body`. Run it once *without* `express.json()` and once *with* it, capturing both `curl -i` outputs. Explain in one sentence why the body differs.
- [ ] **Task 3 — Proxy the app.** Put Nginx in front of your Express app on port 80. Verify with `curl -i`. Then stop the Node process, `curl` again, and explain the status code you get using the three-layer model.
- [ ] **Task 4 (harder) — Trailing-slash trap.** Configure `proxy_pass http://127.0.0.1:3000;` and `proxy_pass http://127.0.0.1:3000/;`, and request `/api/status` through both. Record the path each one delivers to the app, and state the rule the slash controls.
- [ ] **Task 5 (harder) — Static offload.** Serve `/assets/logo.svg` from Nginx (a `location /assets/` block) while `/` proxies to Express. Prove which process served the file using the response headers, then explain why this frees the Node app.

## 📝 Research Journey
- **Why:** I know Linux (`curl`, ports) and I can write JS, but "deploy a server" was a black box — I did not know what a web server *is* underneath, or why tutorials put Nginx in front of Node. The course's 1.5 ("install Nginx/Apache, and build your own server with Express") was the first time I had to connect the two halves I already knew.
- **Struggle:** The word "server" kept meaning three different things at once — the machine, the Nginx process, and my Express app. My first `502` made no sense until I drew the layers and realised Nginx was fine but the app behind it was dead. Similarly, `EADDRINUSE` looked like a code bug until I understood a port is a *resource owned by a process*.
- **Aha moment:** Seeing `curl -i` print the raw status line and headers for the first time. The entire request/response story collapsed from "magic in the browser" into plain text I could read and reason about. A close second: watching a request enter on port 80 and come out of code running on port 3000 — the reverse proxy stopped being an abstraction and became a forwarding rule I could test.
- **Career link:** An AI Engineer who ships a model doesn't hand someone a notebook — they put a service behind a socket and proxy it for the world. The model may be Python, but the *delivery* is exactly this stack: Nginx at the edge, an app server behind it, a database behind that. Knowing which layer fails, and proving it with `curl`, is the difference between shipping and being stuck.

## 🔗 Related Topics
- [`./03-mvc-architecture-in-express.md`](./03-mvc-architecture-in-express.md) — how the Express app you just started is structured internally.
- [`./04-routing-in-express.md`](./04-routing-in-express.md) — the routes and middleware introduced here, in depth.
- [`01-static-dynamic-ssr-vs-csr.md`](./01-static-dynamic-ssr-vs-csr.md) — the *content* this server produces (template HTML vs client-rendered).
- [`../../software_architecture/03-application-architecture-models.md`](../../software_architecture/03-application-architecture-models.md) — where this topology sits in a larger system design.
- [SUGGESTED] `bridge-docker-to-server-topology.md` — bridge note mapping the Nginx→Express→Postgres ports here onto Compose service names, since the course deploys this exact stack with Docker.
- [SUGGESTED] `https-tls-and-certificates.md` — how the `:443` layer works (certificates, Let's Encrypt, termination) — the natural follow-up to Nginx's TLS role.

## 🤔 Open Questions
- [ ] When Nginx sits in front of Express, which layer should own gzip and caching — and how do we avoid double-compressing?
- [ ] How do `proxy_set_header` values (`X-Real-IP`, `X-Forwarded-For`) get trusted safely, and how can a client spoof them if the proxy is misconfigured?
- [ ] For a small course project, is Nginx worth the extra moving part, or is `app.listen(80)` behind Docker's port mapping equivalent in practice?
- [ ] How does `keep-alive` between Nginx and Node change connection reuse and latency compared with a direct connection?

---
*Nothing renders until a socket accepts you. Find the process. Find the port. Read the bytes with `curl`.*

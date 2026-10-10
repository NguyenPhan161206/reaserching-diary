# Deploying the Express SSR + PostgreSQL App with Docker Compose

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Docker`, `#DockerCompose`, `#Deployment`, `#Express`, `#PostgreSQL`, `#DevOps`, `#DevTools`
**References:** Docker Docs "Dockerfile reference" & "Compose file reference", Node.js Docker "Best Practices" (docker/node), PostgreSQL 16 official image (`docker-library/postgres`), nginx "reverse proxy" guide, Let's Encrypt / Certbot docs
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`README.md`](README.md) (images/layers/volumes overview — notes 01–03 are *backlog*, summarised below), [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md), [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md), [`../databases/02-postgresql-core-concepts.md`](../databases/02-postgresql-core-concepts.md)

---

## 📌 Key Takeaways (TL;DR)

- **A `Dockerfile` is a recipe for an immutable snapshot of "app + environment"; `docker compose` is a recipe for the whole system** (web + db + optional nginx). Run the compose file on any machine and you get the same topology.
- **Deployment is the discipline that kills "it worked on my machine"**: the image carries the app *and* its runtime, so your laptop, a VPS, and CI all run the identical bytes.
- **Copy `package*.json` before the source and you get layer caching**: dependency install is reused across rebuilds, turning a 2-minute build into 2 seconds.
- **`depends_on` + a `healthcheck` is what stops "web starts before the DB is ready" crashes.** Port-open is not the same as database-ready.
- **The `pgdata` volume is the only thing that makes data survive `docker compose down`** (and only `down -v` deletes it). The app image is disposable; the volume is not.
- **Never bake secrets into the image.** Env vars come from `.env` (git-ignored) at runtime, not from `ENV` lines in the Dockerfile.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

> **Think of a container as an *immutable snapshot of a running process*, and the Dockerfile as the recipe that bakes it.**

Before any command, hold three ideas:

1. **Image vs container.** An **image** is a frozen, layered filesystem + startup command. A **container** is a running instance of an image. One image → many containers. *(Backlog notes 01–02 cover layers and the build cache in full; here they are prerequisites.)*
2. **Layers and the build cache.** Each `Dockerfile` instruction makes a layer. Docker reuses a cached layer as long as the inputs above it are unchanged — which is why instruction **order** is a performance decision, not a style choice.
3. **Volumes vs the container filesystem.** A container's own filesystem dies with the container. A **volume** is storage that lives *outside* the container lifecycle. *(Backlog note 03 covers volumes/bind mounts.)*

**The mental model for this note — two recipes, one topology:**

```
Dockerfile  =  recipe for ONE thing  →  "app + node + env, snapshot it"
compose.yml =  recipe for a SYSTEM  →  "start db, wait till healthy, start web,
                                        wire them on a network, attach volumes"
```

A single `docker compose up` on a laptop, a VPS, or a CI runner reproduces:

```
                     ┌───────────────────────────────┐
   browser ──:3000──►│ web  (Express SSR, node:20)    │
                     │  /app  ← image                │
                     │  /app/uploads ← volume        │
                     └──────────────┬────────────────┘
                                    │ compose network (DNS: "db")
                                    ▼
                     ┌───────────────────────────────┐
                     │ db  (postgres:16-alpine)       │
                     │  /var/lib/postgresql/data      │
                     │         ← pgdata volume        │
                     └───────────────────────────────┘
```

`web` reaches `db` **by service name** (`db:5432`) — Compose runs a DNS server on its private network. You never hard-code an IP.

> **Why it matters:** the same file that a solo developer runs locally is the file a CI pipeline builds and a VPS runs in production. There is no "deployment script" that drifts from the dev setup; the topology *is* the code.

---

### 2. The Dockerfile for a Node SSR App

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine

WORKDIR /app

# 1) Copy ONLY the dependency manifest first.
#    This layer is reused on every rebuild where dependencies did not change.
COPY package.json package-lock.json ./

# 2) Install EXACT locked versions, production dependencies only.
#    (Older guides write `npm ci --only=production`; npm 7+ renamed the flag
#     to `--omit=dev`. Both mean "skip devDependencies".)
RUN npm ci --omit=dev

# 3) Now copy the source. Changing a .js file only invalidates THIS layer,
#    not the expensive dependency layer above.
COPY --chown=node:node . .

# 4) Give the non-root runtime user a writable upload dir.
#    A named volume mounted here inherits node's ownership on first use.
RUN mkdir -p /app/uploads && chown -R node:node /app/uploads

# 5) Production mode: enables Express view cache and disables dev behaviour.
ENV NODE_ENV=production
EXPOSE 3000

# 6) Drop root. The node image ships a `node` user for exactly this.
USER node

CMD ["node", "src/server.js"]
```

**Why each line exists:**

| Line | Reason |
|------|--------|
| `FROM node:20-alpine` | Small base (~50 MB vs ~1 GB for the full image); pinned major so `node` version is reproducible |
| `WORKDIR /app` | Sets the working directory for all later instructions *and* the runtime — no `cd` needed |
| `COPY package*.json ./` **before** `COPY . .` | **Layer caching.** Dependencies change rarely; source changes constantly. Ordering them this way means a source edit does not trigger `npm ci` again |
| `RUN npm ci --omit=dev` | `npm ci` installs *exactly* `package-lock.json` (reproducible, unlike `npm install`); `--omit=dev` keeps test/build tooling out of the runtime image |
| `COPY --chown=node:node . .` | Non-root user owns the app files, so the process can read them and write where allowed |
| `RUN mkdir -p /app/uploads && chown ...` | The one directory the app writes to at runtime must be writable by `node` |
| `ENV NODE_ENV=production` | Flips Express to production mode (view cache on, stack traces off) — this is what makes §4 of the performance note real |
| `EXPOSE 3000` | Documentation of intent; actual publishing happens in Compose (`ports:`) |
| `USER node` | **Security:** a container running as root that escapes is a root process on the host |
| `CMD ["node","src/server.js"]` | Default startup; exec form so the node process is PID 1 and receives signals (SIGTERM) cleanly |

**`.dockerignore`** — the other half of a fast, safe build. Without it, the build context ships `node_modules`, `.git`, and secrets to the daemon:

```
node_modules
npm-debug.log*
.git
.gitignore
.env
uploads
*.md
Dockerfile
docker-compose.yml
```

`node_modules` must be ignored because `npm ci` runs *inside* the image on the container's platform; copying your host's `node_modules` (possibly native modules built for macOS) breaks the build and bloats the context. `.env` must be ignored so secrets never enter a layer.

---

### 3. `docker-compose.yml` — the Whole System

```yaml
# No `version:` key — it is obsolete in the Compose Specification (Compose v2+).

services:
  web:
    build: .
    ports:
      - "3000:3000"                 # host:container
    env_file:
      - .env                        # POSTGRES_*, SESSION_SECRET, ...
    environment:
      NODE_ENV: production
      DB_HOST: db                   # service name — resolved by compose DNS
      DB_PORT: "5432"
    depends_on:
      db:
        condition: service_healthy  # do NOT start web until the DB is ready
    volumes:
      - uploads:/app/uploads        # user uploads survive restarts
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data   # the data that must survive
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 10
    restart: unless-stopped

volumes:
  pgdata:
  uploads:
```

Notes that matter:

- **No published port on `db`.** Only `web` needs to reach it, and that happens over the internal network. Publishing `5432:5432` invites port conflicts and exposes the database. (See pitfalls.)
- **`depends_on` alone is not enough.** Plain `depends_on` only controls *start order*; Postgres can be started-but-not-yet-accepting-connections. `condition: service_healthy` waits until the `healthcheck` passes, which is the real fix for "web crashes because it connected too early."
- **`restart: unless-stopped`** re-launches a crashed service (and the whole stack after a host reboot), which is what makes it a deployable unit rather than a dev toy.

**Networks:** Compose creates a default network (e.g. `<project>_default`) and attaches every service to it. Service names become DNS names, so `web` connects to `db:5432` with zero configuration. A `networks:` block is only needed when you want to *separate* traffic (e.g. a public network for nginx and a private one for db+web).

---

### 4. Running the Real App

**First-time flow (from the project root, where `docker-compose.yml` lives):**

```bash
# 1) Build images and start the whole system, waiting for db to be healthy
docker compose up --build

# (in another terminal, or with `-d` to detach)
docker compose up -d --build
```

Expected startup order: `db` logs `database system is ready to accept connections`, its healthcheck flips to `healthy`, **only then** `web` starts.

```bash
# 2) Migrations MUST run inside the container, against the container DB.
#    `run --rm` starts a one-off container and deletes it afterwards.
docker compose run --rm web npx knex migrate:latest

# 3) Optional seed data
docker compose run --rm web npx knex seed:run
```

Expected migration output:

```
Using environment: production
Batch 1 run: 3 migrations
```

```bash
# 4) Verify from the host
docker compose ps
docker compose logs web
curl -i localhost:3000
```

Expected `docker compose ps` (both up; `db` healthy):

```
NAME              SERVICE   STATUS                    PORTS
app-db-1          db        Up 2 minutes (healthy)    5432/tcp
app-web-1         web       Up 30 seconds             0.0.0.0:3000->3000/tcp
```

Expected `curl -i`:

```http
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 18422

<!DOCTYPE html> ...
```

**Reloading after a code change:** `docker compose up -d --build web` rebuilds and recreates only the web service; `db` and its volume are untouched.

---

### 5. Data & Secrets

**The volume is the part that persists.** `docker compose down` stops and removes containers and the network, but the named volume `pgdata` (and `uploads`) remain on disk. The *next* `docker compose up` reattaches them and your data is there.

```
docker compose down        → containers gone, pgdata KEPT
docker compose down -v     → containers gone, pgdata DELETED  ⚠️ wipes the database
docker compose up --build  → rebuilds the image; volume untouched
```

That is the key distinction: **rebuilding the app is free; deleting the volume is data loss.** In production, `down -v` should feel dangerous.

**Environment vs config files:**

- **Environment variables** are the right home for per-deployment settings (DB credentials, `SESSION_SECRET`, `NODE_ENV`) because they differ between laptop, staging, and production without changing code.
- **Config files committed to the repo** are for things that are identical everywhere and safe to publish (the `knexfile.js` shape, non-secret defaults).

**Secrets never go in the repo.** The image is a build artifact that may be pushed to a registry and inspected; anything in an `ENV` line or a `COPY`'d file is visible to anyone who can pull it. Instead:

1. Keep a `.env` file **git-ignored** (`.gitignore` must contain `.env`).
2. Commit only a **`.env.example`** documenting the keys with placeholder values.
3. In CI/production, inject secrets from the platform's secret store (GitHub Actions secrets, the VPS's environment, a secret manager), not from a file.

> This matches the repository rule: **no secrets are ever pushed.** If a secret ever lands in a commit, the fix is to rotate it, not just delete the line.

---

### 6. Optional Nginx in Front

Reuse the reverse-proxy knowledge from [`../web_server/02-web-server-nginx-apache-express.md`](../web_server/02-web-server-nginx-apache-express.md). Add a service:

```yaml
  nginx:
    image: nginx:1.27-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
    depends_on:
      - web
    restart: unless-stopped
```

```nginx
# nginx/default.conf
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://web:3000;                 # service name, again
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;  # so Express knows it was HTTPS
    }

    location /static/ {
        proxy_pass http://web:3000;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

With nginx up, remove the host port from `web` (it only needs to be reachable on the internal network; `nginx` owns port 80). **TLS in production:** run Certbot once (`certbot certonly --webroot -w ... -d example.com`) or use its nginx plugin; Certbot writes the certificate into `/etc/letsencrypt`, which the volume above mounts read-only, and renews on a timer. From then on nginx listens on 443 and redirects 80 → 443.

---

### 7. CI & Deploy

**CI (GitHub Actions).** A pipeline typically builds the image in CI (catching broken Dockerfiles *before* deploy) and pushes it to a registry. This reuses the workflow knowledge in [`../../git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md):

```yaml
# .github/workflows/docker.yml (sketch)
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha     # reuse layers between runs
          cache-to: type=gha,mode=max
```

**VPS deploy (the simplest real path):**

```bash
ssh deploy@vps
cd /srv/app
git pull
docker compose up -d --build        # rebuild changed images, restart changed services
docker compose run --rm web npx knex migrate:latest   # if migrations changed
docker image prune -f               # clean dangling images
```

Firewall: expose only **80 and 443** publicly; SSH on a non-default port is a bonus. The database port stays closed to the internet — it is reachable only inside the Compose network.

---

### 8. Command / Code Cheat Sheet

| Command | What it does | When to use | Example / output |
|---------|--------------|-------------|------------------|
| `docker build -t app:1.0 .` | Build an image from a Dockerfile | Testing the build without Compose | `=> exporting to image` |
| `docker compose build` | Build all `build:` services | Before first run / after a Dockerfile change | `Image app-web Built` |
| `docker compose up --build` | Build then start, foreground | First run; watch logs live | services start in order |
| `docker compose up -d --build` | Same, detached | Normal start / redeploy | `Started app-web-1` |
| `docker compose down` | Stop + remove containers/network (keeps volumes) | Shutting down | `Removed...` |
| `docker compose down -v` | **Also deletes volumes** | Only when you want a clean DB | ⚠️ `Removed volume app_pgdata` |
| `docker compose ps` | List services + status + ports | Confirm what is running/healthy | `web Up ... 0.0.0.0:3000->3000` |
| `docker compose logs -f web` | Follow a service's logs | Debugging startup/errors | `> node src/server.js` |
| `docker compose exec web sh` | Shell inside a *running* container | Inspect files, env, process | `/app $` |
| `docker compose run --rm web npx knex migrate:latest` | One-off command in a fresh container, then delete it | Migrations, seeds, scripts | `Batch 1 run: 3 migrations` |
| `docker compose run --rm web npx knex seed:run` | Run seeders | Local/demo data | `Ran 2 seed files` |
| `docker image prune -f` | Remove dangling images | After several rebuilds | `Deleted Images: ...` |
| `docker compose config` | Validate + print the resolved compose file | Before deploying; catch typos | resolved YAML |

---

### 9. Practical Lab — A Real Mini Compose Project

**Goal:** stand up `web` + `db`, run a migration, and get an HTTP 200 on `localhost:3000`.

```bash
mkdir mini-ssr && cd mini-ssr
```

`package.json`:

```json
{
  "name": "mini-ssr",
  "version": "1.0.0",
  "scripts": { "start": "node src/server.js" },
  "dependencies": { "express": "^4.19.2", "pg": "^8.12.0" }
}
```

`src/server.js`:

```js
const express = require('express');
const { Pool } = require('pg');

const app = express();
const pool = new Pool({
  host: process.env.DB_HOST || 'db',
  user: process.env.POSTGRES_USER,
  password: process.env.POSTGRES_PASSWORD,
  database: process.env.POSTGRES_DB,
});

app.get('/', async (req, res) => {
  const { rows } = await pool.query('SELECT now() AS now');
  res.send(`<h1>OK</h1><p>Server time: ${rows[0].now.toISOString()}</p>`);
});

app.listen(3000, () => console.log('listening on :3000'));
```

`.env`:

```
POSTGRES_USER=app
POSTGRES_PASSWORD=secret
POSTGRES_DB=appdb
```

Use the Dockerfile and `docker-compose.yml` from §2–§3 (trim the migration service if you do not have Knex), then:

```bash
docker compose up -d --build
docker compose ps
curl -i localhost:3000
```

**Expected results:**

```
# docker compose ps
NAME       SERVICE  STATUS                    PORTS
app-db-1   db       Up 40 seconds (healthy)    5432/tcp
app-web-1  web      Up 10 seconds             0.0.0.0:3000->3000/tcp

# curl -i localhost:3000
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
...
<h1>OK</h1><p>Server time: 2026-10-10T...</p>
```

Now prove persistence: `docker compose down`, then `docker compose up -d`, and confirm the DB still has its data (the `pgdata` volume survived). Then run `docker compose down -v` and watch it disappear — that is the lesson.

---

### 10. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Mounting a volume over `/app/node_modules` | Dependencies "disappear" or are empty in the container | Never bind-mount `node_modules` over the image's install; on Linux the host dir would shadow the image's |
| `npm install` in production | Non-reproducible builds; lockfile may drift | Use `npm ci` (or `npm ci --omit=dev`) against `package-lock.json` |
| Migration not run inside the container | App connects but tables do not exist | `docker compose run --rm web npx knex migrate:latest` against the *container* DB; the host's `knex` targets the wrong DB |
| Publishing `5432:5432` | "port is already allocated" if a host Postgres runs | Do not publish the DB port; only publish what the outside needs |
| Bind-mount permission errors on a volume dir | `EACCES` when writing uploads / pgdata | Ensure the image dir is owned by the runtime user; on Linux, a bind mount keeps host ownership |
| Forgetting `.dockerignore` | Multi-hundred-MB build context; slow builds | Ignore `node_modules`, `.git`, `.env`, `uploads` |
| `NODE_ENV=development` in production | Templates recompiled on every request (slow); or an edited cached template not showing in dev | Set `NODE_ENV=production` in the image; in development expect no view cache |
| Confusing `CMD` and `ENTRYPOINT` | `docker run image <args>` does not do what you expect | `ENTRYPOINT` is the executable (rarely overridden); `CMD` provides default args (easily overridden) |
| Secrets in `ENV`/`COPY` | Secret visible in `docker history` / image layers | Use runtime env from `.env`/secret store; keep `.env` git-ignored |
| Plain `depends_on` without healthcheck | Web starts before Postgres accepts connections and crashes | `condition: service_healthy` + a `pg_isready` healthcheck |

---

## 🔗 Tech Cross-References

- **[Linux — process & permissions](../linux/01-file-system-paths-and-permissions.md)** — a container *is* a Linux process; `USER node` and volume ownership are the same uid/gid rules you already know from `chmod`/`chown`.
- **[Web server — nginx/Express](../web_server/02-web-server-nginx-apache-express.md)** — the optional nginx service and `proxy_pass http://web:3000` are the reverse proxy from that note, now with a service name instead of an IP.
- **[Databases — PostgreSQL](../databases/02-postgresql-core-concepts.md)** — the `postgres:16-alpine` service is the same engine; `pg_isready` and the `pgdata` volume are the operational half.
- **[Git/GitHub — CI/CD](../../git_github/11-infrastructure-as-code-and-devops-automation.md)** — GitHub Actions builds and pushes the image; the deploy step is `git pull` + `compose up`.
- **[MLOps — deployment architecture](../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md)** — the container image is the unit of deployment for models too; this note's patterns transfer directly to serving an ML service.

## 🧪 Practice Tasks

> No solutions provided — do these, then ask me to grade them (I can make them harder or ask the theory behind them).

- [ ] **Task 1 — Reproduce the lab.** Build the `mini-ssr` project from §9 from scratch, without copying the files. Get `docker compose ps` to show `db (healthy)` and a `curl` HTTP 200. *Done when:* both containers are up and the page shows a timestamp.
- [ ] **Task 2 — Prove persistence.** Seed a row (or use the `now()` query), run `docker compose down`, bring it back up, and show the data survives. Then run `down -v` and show it is gone. Write one sentence explaining *which* command deleted what.
- [ ] **Task 3 — Add nginx.** Put the nginx service from §6 in front of `web`, remove the `web` host port, and confirm `curl localhost` still returns 200 with `X-Forwarded-Proto: http`. *Stretch:* make nginx cache `/static/` and prove a `Cache-Control: immutable` header appears.
- [ ] **Task 4 — Layered rebuild.** Time a full build with `COPY . .` first; then reorder so `package*.json` is copied before the source, and change one `.js` file and rebuild. Show the dependency layer is reused (build time drops). Explain the result in terms of layers.

## 📝 Research Journey

- **Why:** The Web Application Development course ends by "packaging and deploying the SSR app," and the whole point of Docker is exactly the thing I feared most as a beginner — I once had a project that ran on my laptop and crashed instantly on the VPS. `docker compose` is the tool that makes that failure impossible for the class project *and* for every future side project.
- **Struggle:** The two ideas that took longest to click were **layer caching order** and **`depends_on` vs healthcheck**. I thought `depends_on` meant "wait for the DB," and watched `web` crash-on-start for an afternoon before reading that it only orders *starting*, not *readiness*. And I initially copied everything then ran `npm ci`, wondering why every tiny edit re-downloaded all packages.
- **Aha moment:** The `pgdata` volume. Watching `docker compose down` (data safe) and `docker compose down -v` (data gone) back to back made the image/container/volume distinction concrete — the image is disposable, the volume is the thing you are actually protecting. That reframed backups, deploys, and "why did my dev data vanish" in one shot.
- **Career link:** AI engineers ship models in containers — the same `Dockerfile`, the same compose topology, the same volume-vs-image discipline appears in MLOps. Being able to say "the model server is an image, the model weights are a volume, and CI builds the image" is a deployment skill that transfers directly from this course project to production ML.

## 🔗 Related Topics

- [`../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md) — the app being packaged: controllers, models, and views that become `/app` in the image.
- [`../databases/06-knex-query-builder-and-crud.md`](../databases/06-knex-query-builder-and-crud.md) — the migrations/seeds run via `compose run --rm web npx knex ...`.
- [`04-web-performance-caching-and-bottlenecks.md`](../../software_architecture/04-web-performance-caching-and-bottlenecks.md) — why `NODE_ENV=production` and the nginx layer in front matter for latency.
- [SUGGESTED] `06-multi-stage-builds-and-image-size.md` — *why it's important:* once the image gets painful to push, a multi-stage build (build with deps → copy only the runtime artifacts) and Alpine/distroless choices are the next step. It is currently the Low-priority backlog item "image size optimisation," promoted by real deployment cost.
- [SUGGESTED] `07-zero-downtime-deploys-and-rollbacks.md` — *why it's important:* `docker compose up -d --build` has a brief downtime; a real product needs health-gated rollout and a one-command rollback. *Folder:* `dev_foundation/docker/` or `software_architecture/`.

## 🤔 Open Questions

- [ ] For the class project, is a single `web` instance behind nginx enough, or does the grader deploy across multiple instances (which would force Redis-backed sessions)?
- [ ] What is the correct place to run `knex migrate` automatically on deploy without racing multiple `web` replicas?
- [ ] How large is the class project's image, and would a multi-stage build meaningfully reduce the push time to the registry?
- [ ] When does Compose stop being the right tool and Kubernetes/Swarm begin — what concrete signal makes that jump worth it?

---
*"It works on my machine" is not a joke. It is a deployment bug you have not diagnosed yet — and the Dockerfile is the diagnosis.*

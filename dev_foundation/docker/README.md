# Docker — Containers & Orchestration

**Purpose:** Package an application **with the environment it needs**, so it runs the same on your laptop, on a server, and in CI. Learn the container mental model plus the daily CLI workflow.

**Template:** **Template G** for CLI/tooling notes (01–03), **Template A** for the orchestration mental model (04).

## Why this matters

- The classic bug class "works on my machine" disappears when the machine is a container image.
- CI/CD (see [`git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md)) builds and tests inside containers.
- ML pipelines need pinned CUDA/Python environments — `ai_ml/05` depends on this.
- A throwaway PostgreSQL for practicing SQL: `docker run postgres` (see [`../databases/`](../databases/README.md)).

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 05 | Deploy the Express SSR app + PostgreSQL with Compose | G/A | How do I ship the whole web app + database as one reproducible system? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Images, containers, registries, layers | The core mental model; everything else depends on it |
| High | Dockerfile & build workflow | How a `Dockerfile` turns into a runnable image (note 05 uses one, assumes this) |
| High | Volumes, bind mounts, networks, ports | Why data "disappears" and how to fix it |
| Medium | Mental model of orchestration (compose, swarm, k8s — *why* they exist, not how) | Needed before touching any real deployment |
| Low | Image size optimisation (multi-stage builds, `.dockerignore`) | Only when an image becomes painful |
| Low | Debugging containers (`docker logs`, `exec`, `inspect`) | Comes up constantly in practice |

---

## 🔗 Tech Cross-References (Section K)
- **Linux** — a container *is* a Linux process: namespaces (isolated FS/PID/network) + cgroups (CPU/memory limits)
- **YAML** — `docker-compose.yml` is the first YAML file most developers meet
- **Git / GitHub Actions** — CI builds the image from a `Dockerfile` in the repo; bad caching wastes minutes per push
- **Databases** — `docker run postgres` / `mysql` for instant local databases; volumes decide if data survives
- **MLOps** — `ai_ml/05` treats the container image as the unit of deployment
- **Networking** — `-p 5432:5432` is the same idea as exposing a port on a server; DNS between containers

## 🔗 Related Topics
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — the FS a container namespaces
- [`../web_server/README.md`](../web_server/README.md) — the Express SSR app note 05 packages
- [`../databases/02-postgresql-core-concepts.md`](../databases/02-postgresql-core-concepts.md) — the DB that runs in the `db` service
- [`../yaml_config/02-yaml-in-github-actions-and-devops.md`](../yaml_config/02-yaml-in-github-actions-and-devops.md) — compose + CI syntax
- [`../../git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md) — where images get built
- [`../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md) — containers in the ML lifecycle
- [SUGGESTED] `bridge-container-vs-vm.md` — bridge note: what a VM gives you that a container doesn't, and when that still matters
- [SUGGESTED] `bridge-docker-to-databases.md` — the healthcheck + `depends_on` trick note 05 uses, generalised to any multi-service app

---
*Created: 2026-09-28 — Updated: 2026-10-10 — Templates: G (tooling) + A (mental model)*

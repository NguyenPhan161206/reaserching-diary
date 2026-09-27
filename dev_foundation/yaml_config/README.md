# YAML & Config Formats

**Purpose:** Read and write machine-readable configuration correctly — syntax, indentation rules, the traps that silently break CI, and when to choose YAML vs JSON vs TOML vs `.env`.

**Template:** **Template G** for syntax/usage notes (01–02), **Template A** for the "which format when" decision (03).

## Why this matters

- YAML is the first YAML you meet in real work: `.github/workflows/*.yml`, `docker-compose.yml`, Kubernetes manifests, Ansible, Prettier/ESLint config.
- Most YAML bugs are **not syntax errors** — they are wrong types (`on:` parsed as boolean `true`), implicit string coercion (`yes`/`no`/`no`/`1.0`), or indentation mistakes that "work but mean something else".
- Config is code: it deserves the same care as source, including version control and review.

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| — | *(none yet)* | — | — |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | YAML syntax & data types (indentation, anchors, `on`/`yes`/`no` traps) | Prevents the classic silent-failure bugs |
| High | YAML in GitHub Actions & DevOps | Where YAML appears in *my* actual work (CI, compose) |
| Medium | Config format decision: YAML vs JSON vs TOML vs `.env` | Choosing wrongly causes pain later (comments! secrets! parsing cost) |
| Low | Schema validation for config (JSON Schema, `yamllint`) | Only once configs grow beyond a few services |
| Low | Hierarchical config & environment overrides | Later, when a service runs in dev/staging/prod |

---

## 🔗 Tech Cross-References (Section K)
- **GitHub Actions** — CI workflows are YAML; a mis-indented `on:` means your pipeline silently does nothing
- **Docker Compose** — `docker-compose.yml` multi-service orchestration
- **`.env` & secrets** — config and secrets are different concerns; `.env` stays out of git, YAML describes behaviour
- **JSON** — the wire format of APIs (PostgREST returns JSON), same data, different philosophy
- **Python** — `pyproject.toml` is a third syntax; tool config in `package.json` is a fourth
- **Databases** — config files decide connection pooling, and a bad one is a silent performance bug

## 🔗 Related Topics
- [`../docker/`](../docker/README.md) — compose files
- [`../../git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md) — CI/CD workflows in YAML
- [`../databases/05-schema-design-and-data-modeling.md`](../databases/05-schema-design-and-data-modeling.md) — config for DB connection/pooling
- [`../../software_architecture/01-event-driven-architecture-task-orchestration.md`](../../software_architecture/01-event-driven-architecture-task-orchestration.md) — config expresses architecture decisions
- [SUGGESTED] `bridge-config-is-code.md` — bridge note: config in version control, review, and testing

---
*Created: 2026-09-28 — Planned templates: G (syntax/usage) + A (format decision)*

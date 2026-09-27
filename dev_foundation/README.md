# Dev Foundation — Engineering Fundamentals

**Purpose:** The practical engineering layer that everything else in this diary is built on. Linux commands, GitHub workflows, containers, config formats, web rendering mechanisms, and databases — studied at **working depth** (I can use them, debug them, and read other people's code with them), not at reference-book depth.

**Why this folder exists:** notes in `ai_ml/`, `iot_aiot/`, and `software_architecture/` were explaining *systems* while the *tools* used to build those systems were assumed knowledge. This folder closes that gap. Every note here follows **Template G (Dev Tool & Command Note)** or **Template A (Tech Note)** and must cross-reference other technologies (AGENTS.md Section K).

> **Language scope:** these technologies are learned "sơ" (overview → hands-on → later deepen when a real project demands it). The goal is a working mental model + muscle memory on commands, not exhaustive coverage.

---

## 📚 Sub-folders

| Folder | Topic | Template | Core question |
|--------|-------|----------|---------------|
| [`linux/`](linux/README.md) | Shell, filesystem, processes, permissions | G | How does the machine under the terminal actually work? |
| [`docker/`](docker/README.md) | Images, containers, volumes, networks, orchestration | G + A | How do I ship an environment, not just code? |
| [`yaml_config/`](yaml_config/README.md) | YAML syntax, config design, CI usage | G + A | How do I express machine-readable configuration correctly? |
| [`web_basics/`](web_basics/README.md) | HTML tags, CSS layout, JS core, DOM & event loop | A + G | How does a browser turn text into pixels and respond to you? |
| [`databases/`](databases/README.md) | Relational model, SQL, PostgreSQL, MySQL, PostgREST | G + A | Where does state live, and how do I query it safely? |
| [`dsa/`](dsa/README.md) | **Anchor only** — roadmap + link | Flexible | Where do I write the algorithm code? → **other repo** |

---

## 🗺️ Learning Path (order matters)

```
Phase 0 — already done
  └─ git_github/            (11 notes — Git & GitHub fundamentals)   [PREREQUISITE]

Phase 1 — the tool you touch every single day
  └─ linux/                 ← START HERE
     (files & permissions → navigation → pipes → processes/services)

Phase 2 — the practice track (runs in PARALLEL with Phase 1)
  └─ dsa/ (anchor)  ──────────────►  DSA-reserching-diary  [code repo]
     Learn algorithms by writing code, using the Linux CLI daily to
     read logs, run tests, and manage version control.

Phase 3 — package & ship the environment
  └─ docker/
  └─ yaml_config/           (feeds GitHub Actions in git_github/11)

Phase 4 — read (and write) a code base
  └─ web_basics/            (mechanism-first: tags → layout → cascade → JS → DOM)
  └─ databases/             (SQL → PostgreSQL → MySQL vs PG → PostgREST → schema design)

Phase 5 — build (with a real project)
  └─ Joblink / UMT-Truck-Blind-Sight / Snake_ANN repos
     Every gap found while building goes back into this folder as a note.
```

**Rule of thumb:** when a concept from `ai_ml/`, `iot_aiot/`, or `software_architecture/` needs a tool to be real, it gets a note here — and that note links back to the system note that demanded it.

---

## 🔗 How the technologies connect to each other

> Deliberately shallow — just enough to make each note make sense in context. Depth comes later, per technology.

| If you are learning... | You will touch... | Because... |
|------------------------|-------------------|------------|
| **Linux** | Git | Every Git object is a file; a repo is a directory of files + history |
| **Linux** | Docker | A container is just a Linux process with restricted view of the FS |
| **Linux** | YAML | Config files (`docker-compose.yml`, `.github/workflows/*.yml`) are files you read with `cat` |
| **Linux** | DSA | Reading compiler/test output with `grep`, `|`, `less` is half of debugging algorithms |
| **Docker** | GitHub Actions | CI builds your image from a `Dockerfile` sitting in the repo |
| **Docker** | Databases | `docker run postgres` is how you get a throwaway database in 3 seconds |
| **YAML** | JSON / `.env` | Same job, different trade-offs (indentation vs. braces vs. secret safety) |
| **YAML** | GitHub Actions | The single most common place beginners meet YAML in real work |
| **HTML/CSS** | JavaScript | HTML = structure, CSS = presentation, JS = behaviour, DOM = the bridge |
| **HTML/CSS** | Databases | Every list/table you see is data rendered by a template |
| **JavaScript** | PostgREST | `fetch()` talks to a PostgREST endpoint and receives JSON |
| **Databases** | PostgREST | Your database gets an instant REST API — no hand-written server |
| **Databases** | DSA | Tables, indexes, and B-trees are data structures with a cost model |
| **Databases** | Git | Transactions and commits are the *same idea* applied to data vs. files |

---

## 📋 Note Index

| Folder | Notes written | Notes planned |
|--------|---------------|---------------|
| `linux/` | 2 | 5 |
| `docker/` | 0 | 4 |
| `yaml_config/` | 0 | 3 |
| `web_basics/` | 0 | 5 |
| `databases/` | 0 | 5 |
| `dsa/` | 2 (anchor + roadmap) | — |

Each sub-folder README keeps its own **Note Index** and **Note Backlog** table (AGENTS.md Section M).

---

## 🔗 Related Topics
- [`git_github/`](../git_github/README.md) — prerequisite track; Git internals assume Linux knowledge
- [`software_architecture/`](../software_architecture/README.md) — the "why" these tools exist
- [`ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md) — Docker + YAML in a real ML pipeline
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- 🔗 [Joblink](https://github.com/qminh77/joblink) — web + database + auth, the Phase 5 target

## 🤔 Open Questions
- [ ] Should Linux notes be written for Ubuntu 24.04 specifically (the actual environment) or kept distribution-agnostic?
- [ ] Where does shell scripting (Bash) fit — a dedicated `linux/06-bash-scripting.md`, or only when a real task needs it?
- [ ] Do we need a `dev_foundation/tooling/` folder later for things like `curl`, `jq`, `tmux`, `make`?

---
*Foundation first. Theory is cheap when you can run it.*

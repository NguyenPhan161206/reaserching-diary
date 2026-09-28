# Linux — Shell, Filesystem, Processes

**Purpose:** The operating system under every other tool in this diary. Learn to *navigate, inspect, control, and automate* a Linux machine from the terminal.

**Template:** **Template G (Dev Tool & Command Note)** for every note here. See `../../NOTE_TEMPLATE.md`.

**Environment:** Ubuntu (WSL2 or native) — commands are distribution-agnostic POSIX/Linux unless noted.

## Why start here

- Every other tool in `dev_foundation/` runs inside a Linux machine or container.
- Git is a folder of files; Docker is a Linux process; `git log` is text processing.
- The DSA track (see [`../dsa/`](../dsa/README.md)) needs the CLI daily: reading compiler errors, piping test output, checking memory usage.

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | [File System, Paths & Permissions](01-file-system-paths-and-permissions.md) | G | What is actually on disk, and who is allowed to touch it? |
| 02 | [Navigation & Core CLI](02-navigation-and-core-cli.md) | G | How do I move, search, and read documentation without the mouse? |
| 03 | [Text Processing, Pipes & Redirects](03-text-processing-and-pipes.md) | G | How do commands compose into a pipeline? |
| 04 | *(planned)* Processes, Services & Package Manager | G | What is running, how do I start/stop it, where did it come from? |
| 05 | *(planned)* Shell Environment & Aliases | G | What does the shell actually read at startup? |
| 06 | *(planned)* Remote & Network CLI | G | How do I reach a machine or an API from the terminal, safely? |
| 07 | *(planned)* Bash Scripting | G | When does a pipeline need conditionals, functions, and `set -euo pipefail`? |
| 08 | *(planned)* Disk & Memory Diagnostics | G | Why is this slow, and why is this huge? |
| 09 | *(planned)* Text Editors — `vim` Basics | G | The editor that is a hard requirement on most servers |

**Progress: 3 / 9 written.** The numbered index is the *plan*; only ideas without a slot yet live in the backlog below.

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| Medium | Bridge: CLI for ML datasets (`find`/`awk`/`jq` over JSONL, CSV) | Connects this track to `ai_ml/` data pipelines — the first place `awk` earns its keep in the ML stack |
| Low | Terminal multiplexing (`tmux`) | Split panes for editor + test runner; note 03's `less` is the single-window version of the same idea |
| Low | Text user interfaces (`less` navigation deep dive, `dialog`) | Only when a script needs to ask the operator something |

---

## 🔗 Tech Cross-References (Section K)
- **Git** — every commit is a file in `.git/`; permissions and line endings (`CRLF`) are the #1 Windows-Linux bug source
- **Docker** — a container is a Linux process with namespaced filesystem + cgroup limits
- **YAML** — config files you inspect with `cat`/`less` (`docker-compose.yml`, `.github/workflows/*.yml`)
- **Shell vs PowerShell** — same goal, different philosophy: pipes + composability vs. objects + .NET
- **DSA** — `grep`, `|`, `less` are how you read compiler errors and judge algorithm output
- **Databases** — `psql`, `mysql` clients are just terminal programs talking over TCP

## 🔗 Related Topics
- [`git_github/01-version-control-concepts.md`](../../git_github/01-version-control-concepts.md) — Git's three areas are directory entries on disk
- [`git_github/03-core-daily-workflow.md`](../../git_github/03-core-daily-workflow.md) — the daily commands built on these notes
- [`../docker/`](../docker/README.md) — containers inherit the same filesystem model
- [`../dsa/01-learning-roadmap.md`](../dsa/01-learning-roadmap.md) — where the CLI gets used daily
- [`../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md) — the measurement half of that loop: `time`, ratio testing, peak RSS
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `bridge-cli-for-ml-datasets.md` — `find`/`awk`/`jq` over JSONL/CSV dataset files → `ai_ml/` data pipelines

---
*Created: 2026-09-28 — Template: G (Dev Tool & Command Note)*

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
| 03 | [Text Processing, Pipes & Redirects](03-text-processing-and-pipes.md) | G | How do commands compose into a pipeline? |
| 02 | *(planned)* Navigation & core CLI | G | Move around efficiently without the mouse |
| 04 | *(planned)* Processes, Services & Package Manager | G | What is running, how do I start/stop it, where did it come from? |
| 05 | *(planned)* Shell Environment & Aliases | G | What does the shell actually read at startup? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Navigation & core CLI (`cd`/`ls`/`find`/`locate`, `man`, `--help`) | Everyday friction; needed before anything else is comfortable |
| High | Processes, services, packages (`ps`, `top`, `kill`, `systemctl`, `apt`) | Docker and dev servers both require this |
| Medium | Shell environment (`.bashrc`, `PATH`, `alias`, `export`) | Explains "command not found" and env-var bugs |
| Medium | `curl` + `wget` + `ssh` + `scp` | Talking to remote machines and APIs from the terminal |
| Low | Bash scripting (loops, functions, `set -euo pipefail`) | Only when a repetitive task appears |
| Low | Disk & memory diagnostics (`du`, `df`, `free`, `strace` intro) | Debugging "why is this slow/huge" |
| Low | Text editors: `vim` basics | Vim is an environment requirement on most servers |

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
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `terminal-multiplexing.md` (tmux) — split panes for editor + test runner; belongs in `linux/`

---
*Created: 2026-09-28 — Template: G (Dev Tool & Command Note)*

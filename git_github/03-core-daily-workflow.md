# The Core Daily Workflow

**Created Date:** 2026-09-15
**Tags:** `#Git` `#Workflow` `#Daily` `#Status` `#Commit` `#Diff` `#Gitignore`
**References:** [Pro Git — Daily Git](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
**Status:** Published
**Difficulty:** Beginner

---

## 📌 Key Takeaways (TL;DR)

- `status` → `add` → `commit` → `diff` → `log` covers 80% of daily Git usage
- `git add -p` stages individual hunks — the key to atomic commits
- Commit messages: imperative mood, <50 chars subject, explain **what + why** not **how**
- `.gitignore` should be committed in the very first commit
- **Never commit secrets** — once pushed, assume they're public forever

---

## 🧠 Detailed Notes

### 1. File Lifecycle

A file is in exactly one state:

| Family | State | Meaning |
|--------|-------|---------|
| Untracked | — | New file, Git has never recorded it |
| Tracked | Unmodified | Committed, not changed since |
| Tracked | Modified | Changed but not staged |
| Tracked | Staged | Added to staging area, ready to commit |

Goal of a session: move files from modified → staged → committed, repeat.

### 2. Reading State: `git status`

```bash
git status --short
 M app.py        # modified, not staged
A  README.md     # staged (added)
?? notes.txt     # untracked
```

Two letters per line: **(staged status)(working-tree status)**. Space = no change, `M` = modified, `A` = added, `D` = deleted, `??` = untracked. A file both staged and modified shows as `AM`.

### 3. Staging Changes: `git add`

```bash
git add app.py               # one file
git add app.py tests/        # file + directory
git add *.py                 # shell glob
git add -A                   # everything (repo-wide, predictable)
```

**Stage hunks interactively** — the killer feature for atomic commits:
```bash
git add -p app.py
# Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]?
```
- `y` = stage, `n` = skip, `s` = split, `e` = edit manually

**Unstage:** `git restore --staged app.py`

> **Warning:** `git add -A` then `git commit` is the fastest way to produce a sloppy commit that mixes unrelated changes. Prefer staging deliberately.

### 4. Recording a Commit: `git commit`

**Anatomy of a good commit message:**
```
fix: handle empty cart at checkout

The cart total now defaults to zero when the cart is empty,
instead of raising a KeyError that took down the order page.

Closes #142
```

- Subject: imperative, <50 chars, summarizes **what + why**
- Conventions: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:` (Conventional Commits)
- Body: explain context for future readers debugging at 2am

**Short vs full:**
```bash
git commit -m "fix: handle empty cart at checkout"
git commit                 # opens editor for subject + body
```

**Amend last commit:**
```bash
git add forgotten.py
git commit --amend --no-edit   # fold into last commit, same message
```

> **Warning:** `amend` replaces the last commit with a new one. Fine on branches you own, but never amend a commit already pushed to a shared branch.

**Atomic commits:** Self-consistent (doesn't break build), changes one logical thing, message explains it.

### 5. Inspecting Changes: `git diff`

```bash
git diff              # working tree vs staging (unstaged)
git diff --cached     # staging vs last commit (what you'll commit)
git diff --stat       # summary of files + line counts
git diff --word-diff  # highlights changed words, great for prose
```

**Healthy habit:**
1. `git diff` — review working-tree edits
2. `git add` — stage deliberately
3. `git diff --cached` — confirm staged snapshot
4. `git commit`

### 6. The `.gitignore` File

**Pattern syntax:**
- `node_modules/` — directory (trailing slash), anywhere in tree
- `*.pyc` — any file matching glob
- `/build/` — leading slash anchors to repo root
- `!keep.txt` — re-include exception (last match wins)
- `**/logs/` — `**` matches any number of directories

**Minimal practical .gitignore:**
```gitignore
# dependencies / build output
node_modules/
dist/
vendor/
*.pyc
__pycache__/

# secrets
.env
.env.local

# editors / OS
.DS_Store
.vscode/
.idea/
```

Generate tailored to your stack at [gitignore.io](https://gitignore.io).

**Force-add:** `git add -f file` bypasses ignore rules (use deliberately, content must not be sensitive).

> **Tip:** Commit `.gitignore` in the very first commit — it's part of the repo's contract.

### 7. Reading History: `git log`

```bash
git log --oneline --graph --decorate --all
* 9fceb02 (HEAD -> main, origin/main) fix: handle empty cart
* 41bb2cd feat: add login page
| * d3e11f2 (feature/checkout) wip: payment widget
|/
* b91a7d4 chore: init project
```

**Filtering:**
```bash
git log --oneline -3 --author="ana" --since="2 weeks ago"
git log --oneline -- src/app.py          # commits touching a file
git log -S "functionName" --oneline      # when did this string appear?
```

### 8. The Working Rhythm

1. `git status` — know where you are
2. Edit files in working tree
3. `git diff` — review changes
4. `git add` — stage deliberately
5. `git commit` — clear message
6. `git log --oneline` — confirm history

> **Tip:** Commit small, commit often. A commit you can describe in one sentence is one you can reason about, revert cleanly, and bisect for bugs.

> **Danger:** Never commit secrets — API keys, passwords, `.env` files. Once pushed, assume public forever.

---

## 📝 Research Journey

- **Why:** This is the chapter that matters most — the daily workflow is where Git lives or dies in practice
- **Struggle:** Understanding `git add -p` (hunk staging) felt complex, but it's the key to clean history
- **Aha moment:** The diff → add → diff --cached → commit rhythm makes you deliberate about every commit
- **Career link:** Clean commit history is crucial for ML projects — version controlling model changes, data pipelines, and experiment configurations

---

## 🔗 Related Topics

- [[git_github/02-getting-started.md]] — previous: setup
- [[git_github/04-branching-and-merging.md]] — next: branches
- [[git_github/06-undoing-and-recovery.md]] — undoing mistakes
- [[git_github/10-workflows-cookbook.md]] — Conventional Commits conventions

---

## 🤔 Open Questions

- [ ] How does `git stash` interact with the staging area vs working tree?
- [ ] What's the best `.gitignore` strategy for monorepos with multiple languages?

---
*Source: git-github-practical/chapters/ch03-core-workflow.tex*

# Working with Remotes: Push, Pull, Fetch

**Created Date:** 2026-09-15
**Tags:** `#Git` `#GitHub` `#Remote` `#Push` `#Pull` `#Fetch` `#Collaboration`
**References:** [Pro Git — Remotes](https://git-scm.com/book/en/v2/Git-Basic-Branching-and-Merging)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- A remote is just another copy of the repo — usually GitHub — not "the real" one
- `fetch` downloads without touching your work; `pull` = fetch + integrate; `push` publishes
- **Push rejected?** Someone else pushed first — pull, resolve, push again
- `git push -u origin main` — first push sets the upstream, then just `git push`
- **Never force-push shared history** — use `--force-with-lease` only on branches you own
- **Golden rules:** Pull before you start; push as soon as a logical unit is done

---

## 🧠 Detailed Notes

### 1. What a Remote Is

A **remote** is a second copy of your repository, typically on GitHub. It's a peer — not more authoritative. Three commands manage the exchange:

| Command | What it does |
|---------|-------------|
| `fetch` | Download remote's new commits — **does not touch your work** |
| `pull` | fetch + integrate changes into current branch |
| `push` | Publish local commits to the remote |

The name `origin` is convention for "the remote I cloned from" — not special, just default.

### 2. Inspecting and Managing Remotes

```bash
git remote -v
# origin  git@github.com:user/repo.git (fetch)
# origin  git@github.com:user/repo.git (push)

git remote add upstream https://github.com/original/repo.git
git remote rename upstream parent
git remote remove origin
```

**HTTPS vs SSH:** Both work. HTTPS = token/credential helper; SSH = key-based (no password prompts). Pick SSH if you prefer no prompts.

### 3. Fetch, Pull, Push — One Mental Model

**Fetch (always safe):**
```bash
git fetch origin
git log --oneline main..origin/main     # what's new on remote?
```

Updates remote-tracking branches (`origin/main`). Your working tree and local branches are untouched.

**Pull (can conflict):**
```bash
git pull                    # uses configured strategy
git pull --rebase           # replay your commits on top (linear history)
```

When in doubt, `fetch` first and look, then integrate deliberately.

**Push (has consequences):**
```bash
git push
git push origin feature/login
```

The only command that changes the remote — the one with consequences for others.

### 4. Tracking Branches

A **tracking (upstream) branch** pairs your local branch with a remote one. Once set, plain `git push` and `git pull` know where to go.

```bash
git push -u origin main    # first push: set upstream
# After that, just:
git push
git pull
```

### 5. The Daily Collaboration Cycle

1. `git pull` at the start of a session
2. Work and commit locally
3. `git push`
4. If push rejected — someone else pushed first — pull, resolve, retry

**Two golden habits:** Pull before you start; push as soon as a logical unit is done. Small, frequent syncs = tiny conflicts.

### 6. Handling Divergence

**Why pushes get rejected:** Git refuses if the remote has commits you don't have locally — overwriting would destroy someone's work. "Non-fast-forward" rejection.

**Pull with merge vs rebase:**

| Strategy | Result | History |
|----------|--------|---------|
| Pull with merge (default) | Creates merge commit combining both lines | Shows "merge bubble" |
| Pull with rebase | Lifts your commits, fetches remote's, replays yours on top | Clean linear history |

Set preference: `git config --global pull.rebase true` (recommended for linear history).

**Conflict during pull/rebase:** Resolve as usual, then `git rebase --continue` (or `git merge --continue`). Bail: `git rebase --abort`.

### 7. Force-Pushing: When and When Never

**`--force-with-lease` is your only friend.** Plain `--force` silently overwrites; `--force-with-lease` checks that the remote looks the way you expect first.

**Golden rule:** You may rewrite your **own unpublished commits**; never rewrite commits others rely on.

> **Danger:** `git push --force` overwrites the remote and can silently destroy a teammate's commits. Use `--force-with-lease` only on feature branches you own.

### 8. Common Scenarios

**Contribute to a fork:**
```bash
git clone git@github.com:you/repo.git
cd repo
git remote add upstream git@github.com:them/repo.git
git fetch upstream
```
Push to `origin` (your fork), open PR against `upstream`.

**Delete remote branch:** `git push origin --delete feature/login`

---

## 📝 Research Journey

- **Why:** Remotes are where local Git meets team collaboration — understanding fetch vs pull vs push is critical
- **Struggle:** The concept that `origin` is "not special" was confusing at first — it felt like the real repository
- **Aha moment:** Realizing every clone is a peer made me stop treating GitHub as "the source" and start thinking in terms of sync
- **Career link:** Open-source contributions require the fork → clone → PR workflow — essential for building a public AI Engineer portfolio

---

## 🔗 Related Topics

- [[git_github/04-branching-and-merging.md]] — previous: branches
- [[git_github/06-undoing-and-recovery.md]] — next: undoing
- [[git_github/09-pull-requests-code-review.md]] — PR workflow
- [[git_github/08-github-platform.md]] — GitHub features

---

## 🤔 Open Questions

- [ ] How does `git pull --rebase` handle complex multi-commit divergences?
- [ ] When is merge strategy preferable over rebase for team history?

---
*Source: git-github-practical/chapters/ch05-remotes.tex*

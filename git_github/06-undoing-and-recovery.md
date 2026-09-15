# Undoing and Recovery

**Created Date:** 2026-09-15
**Tags:** `#Git` `#Undo` `#Recovery` `#Reset` `#Revert` `#Stash` `#Reflog`
**References:** [Pro Git — Undoing Things](https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- The difference between beginner and pro is knowing what to do when something goes wrong
- **Only two truly destructive commands:** `git reset --hard` + `git clean` — everything else is recoverable
- **Uncommitted edits?** `git stash` (keep a copy) or `git restore` (discard)
- **Commit not pushed?** `git commit --amend`, `reset --soft`, or `reset --hard`
- **Already pushed to shared branch?** `git revert` — never force-push
- **Work looks gone?** `git reflog` first — it can resurrect almost anything

---

## 🧠 Detailed Notes

### 1. The Undo Map

| Situation | Command | What it does |
|-----------|---------|-------------|
| Uncommitted edit | `git restore file` | Discard working-tree changes |
| Staged wrong file | `git restore --staged file` | Unstage, keep edits |
| Committed, not pushed | `git reset --soft HEAD~1` | Undo commit, keep changes staged |
| Committed, not pushed, scrap it | `git reset --hard HEAD~1` | Delete commit + changes |
| Pushed to shared branch | `git revert` | New commit that undoes old one |
| Changed mind, not lost | `git stash` | Park changes, restore later |
| Gone entirely | `git reflog` + `reset` | Resurrect lost commits |

### 2. Discarding Working-Tree Changes

```bash
git restore app.py            # undo edits in one file
git restore .                 # undo edits everywhere (careful!)
```

Throws away uncommitted edits, reverting to last commit. Does not touch staging area (unless `--staged`) or history.

> **Warning:** Once discarded with `restore`, those edits are gone — no undo for the undo. If unsure, `git stash` instead.

**Clean untracked files:**
```bash
git clean -n          # dry run: list what would be deleted
git clean -fd         # delete untracked files + directories
```

> **Danger:** `git clean -fd` deletes permanently. Always check with `-n` first.

### 3. Unstaging and Uncommitting

**Unstage:**
```bash
git restore --staged app.py
```

**Reset — three flavors:**

| Flavor | Branch label | Staging area | Working tree |
|--------|-------------|-------------|-------------|
| `--soft` | Moves | Stays staged | Untouched |
| `--mixed` (default) | Moves | Unstaged | Untouched |
| `--hard` | Moves | Discarded | Discarded |

```bash
git reset --soft HEAD~1       # undo commit, keep staged
git reset HEAD~1              # undo commit, keep working edits
git reset --hard HEAD~1       # undo commit, discard changes
git reset --hard 41bb2cd      # jump anywhere
```

### 4. Safe Undo for Shared History: `git revert`

`reset` **moves** history — unsafe once others have your commits. `revert` **adds** a new commit that applies the reverse. History stays intact and append-only.

```bash
git revert 41bb2cd                 # undo one commit
git revert --no-edit HEAD          # undo latest commit
git revert -m 1 HEAD               # undo merge commit (keep main line)
```

Reverting a merge needs `-m 1` to tell Git which parent line to keep — a common gotcha.

### 5. Stashing Work in Progress

Parks your changes aside and restores them later — perfect for context switches.

```bash
git stash                      # save and clean working tree
git stash push -m "wip: cart"  # with a label
git stash list                 # what's parked
git stash pop                  # restore latest and remove it
git stash apply                # restore but keep the stash
git stash drop                 # discard a stash
```

- **Include untracked:** `git stash -u`
- **Turn stash into branch:** `git stash branch feature/login` — creates branch, applies stash, drops it from list

> **Tip:** Always label stashes (`-m`). Weeks-old anonymous stashes are unreadable.

### 6. The Reflog: Git's Safety Net

The **reflog** is a journal of **every move of every branch label** — commits, resets, rebases, merges, stash operations. Local to your machine.

```bash
git reflog
# f5b3c2e (HEAD -> main) HEAD@{0}: commit: fix: price rounding
# b91a7d4 HEAD@{1}: commit: wip experiment
# 41bb2cd HEAD@{2}: commit: feat: login page

git reset --hard b91a7d4    # travel back to wip experiment
git branch rescued f5b3c2e  # resurrect a commit as a branch
```

**Limits:** Reflog only records operations on **this machine**. Entries expire after ~90 days. A commit you never made, in a clone you never had, can't be recovered. That's why you push.

> **Tip:** There are only a handful of truly destructive Git commands: `git reset --hard` (with `clean`) and `git clean`. Everything else is recoverable via reflog.

### 7. Decision Tree

1. **Uncommitted edits** you want to keep? → `stash` or commit
2. **Uncommitted edits** you want to lose? → `restore` (tracked) / `clean` (untracked)
3. **Commit not pushed**, wrong commit? → `amend`, `reset --soft`, or `reset --hard`
4. **Commit already pushed** to shared branch? → `revert`. Never force-push
5. **Work looks gone?** → `reflog` before anything else

---

## 📝 Research Journey

- **Why:** Knowing how to undo mistakes is what separates Git users from Git pros — this is the safety net
- **Struggle:** The three flavors of `reset` (--soft, --mixed, --hard) were confusing until I mapped them to what gets affected
- **Aha moment:** The reflog is like a time machine — almost nothing is truly lost as long as you haven't done `git clean -fd`
- **Career link:** In ML projects, you often need to revert model changes or recover lost experiment configs — `revert` and `reflog` are lifesavers

---

## 🔗 Related Topics

- [[git_github/05-remotes-push-pull-fetch.md]] — previous: remotes
- [[git_github/07-tags-and-releases.md]] — next: tags
- [[git_github/03-core-daily-workflow.md]] — commit workflow
- [[git_github/10-workflows-cookbook.md]] — 15 everyday undo scenarios

---

## 🤔 Open Questions

- [ ] How does `git stash` handle binary files and large datasets?
- [ ] What's the safest way to remove a committed secret from shared history?

---
*Source: git-github-practical/chapters/ch06-undoing-recovery.tex*

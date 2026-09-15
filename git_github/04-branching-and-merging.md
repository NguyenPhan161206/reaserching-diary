# Branching and Merging

**Created Date:** 2026-09-15
**Tags:** `#Git` `#Branching` `#Merging` `#Conflicts` `#Collaboration`
**References:** [Pro Git — Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- A branch is a **named pointer to one commit** — creating one is instant
- `git switch -c name` creates and moves to a branch in one step
- **Fast-forward merge:** target branch hasn't moved → just slide the label
- **True merge:** both branches diverged → creates a merge commit with two parents
- Conflicts are **not dangerous** — they're Git refusing to silently destroy either side's work
- Keep `main` always releasable; work on small, short-lived branches

---

## 🧠 Detailed Notes

### 1. Branch Basics

```bash
git branch feature/login      # create
git switch feature/login      # switch
git switch -c feature/login   # create + switch in one step
git switch -c feature/login main   # create from a specific point
```

The current branch is where HEAD points. Every commit advances **only** the current branch's label; all others stay untouched.

> **Tip:** Use `git switch` for moving between branches, `git branch` for creating/deleting. `checkout` still works but `switch` communicates intent clearly.

### 2. Branch Operations

**List branches:**
```bash
git branch -vv          # local branches, last commit, upstream
git branch -a           # include remote-tracking branches
git branch --merged     # branches already merged (safe to delete)
git branch --no-merged  # branches with unmerged work
```

**Rename:** `git branch -m old-name new-name`

**Delete:**
```bash
git branch -d feature/login      # safe — refuses if unmerged
git branch -D feature/login      # force — lose the work!
```

**Detached HEAD:** When HEAD points directly at a commit instead of a branch. Happens with `git switch <hash>`. Commits made here belong to no branch and will be lost when you switch away. Fix: `git switch -c save-my-work`.

### 3. Merging: Combining Work

**Two kinds of merge:**

| Type | When | Result |
|------|------|--------|
| **Fast-forward** | Target branch hasn't moved since branch was created | Git slides the branch label forward. History stays linear |
| **True merge** | Both branches have new commits | Creates a merge commit with two parents |

```bash
git switch main
git merge feature/login
# Updating ... Fast-forward
```

**Force a merge commit:** `git merge --no-ff` — creates a visible "feature landed" node even when fast-forward is possible.

**Merge strategies you'll meet:**
- `ort` — modern default, three-way merge
- `ours`/`theirs` — mechanical resolution for special cases
- `octopus` — merges more than two heads (rare)

### 4. Merge Conflicts

**What a conflict is:** Two branches changed the **same lines** in **different ways**. Git can't guess which version is correct, so it stops and asks you.

**A conflict is NOT a mistake.** It's Git refusing to silently destroy either side's work.

**Reading conflict markers:**
```
<<<<<<< HEAD
console.log("legacy behavior");
=======
console.log("new behavior");
>>>>>>> feature/refactor
```

- `<<<<<<< HEAD` = your version (what you're merging into)
- Part before `=======` = yours
- Part after `=======` = theirs (branch being merged)
- `>>>>>>> feature/refactor` = end marker

**Resolving:**
1. Open each conflicted file, locate markers
2. Edit until correct, remove all markers
3. `git add` the file — staging tells Git "resolved"
4. When nothing is left marked: `git commit`

**VS Code:** Highlights conflicts inline with colored buttons: Accept Current / Accept Incoming / Accept Both. Click through, save, commit.

**Abort:** `git merge --abort` — restores everything to before the merge.

### 5. Hands-On Scenario

1. On `main`, change line 1 of `config.py`, commit
2. Create `feature/x`, change line 1 differently, commit
3. `git switch main` and `git merge feature/x`
4. Git reports `CONFLICT` and names the file
5. Open file, resolve hunk, `git add config.py`
6. `git commit` to seal the merge
7. `git log --oneline --graph --all` — see merge node with two parents

### 6. When to Branch

- Branch per feature or fix — small, short-lived
- Keep `main` always releasable; merge in small increments
- Long-lived branches accumulate drift and conflicts — avoid
- Don't branch for a one-line typo on a solo project; **do** branch for anything you want to review or might abandon

> **Tip:** Conflicts are normal and almost never destructive. If lost, `git merge --abort` or `git status` — Git tells you exactly which files need attention.

---

## 📝 Research Journey

- **Why:** Branching is where Git earns its reputation — the mechanism for parallel work without chaos
- **Struggle:** Understanding the difference between fast-forward and true merge took visualizing the commit graph
- **Aha moment:** Once I understood branches are just pointers, everything clicked — merging is just moving pointers around
- **Career link:** ML teams need clean branching for experiment isolation — model A vs model B, feature branches for data pipelines

---

## 🔗 Related Topics

- [[git_github/03-core-daily-workflow.md]] — previous: daily workflow
- [[git_github/05-remotes-push-pull-fetch.md]] — next: remote collaboration
- [[git_github/09-pull-requests-code-review.md]] — PR workflow with branches
- [[git_github/06-undoing-and-recovery.md]] — undoing merge mistakes

---

## 🤔 Open Questions

- [ ] When should you prefer `git merge --no-ff` over letting fast-forward happen?
- [ ] How do rebase-and-merge vs merge-commit affect `git bisect` and `git blame`?

---
*Source: git-github-practical/chapters/ch04-branching-merging.tex*

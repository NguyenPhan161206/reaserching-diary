# Workflows, Best Practices, and the Cookbook

**Created Date:** 2026-09-15
**Tags:** `#Git` `#GitHub` `#Workflows` `#GitFlow` `#GitHubFlow` `#TrunkBased` `#ConventionalCommits` `#BestPractices`
**References:** [Conventional Commits](https://www.conventionalcommits.org), [Trunk-Based Development](https://trunkbaseddevelopment.com)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- Three proven workflows: **Git Flow** (release trains), **GitHub Flow** (continuous delivery), **Trunk-Based** (small teams, feature flags)
- **Conventional Commits** prefixes (`feat:`, `fix:`, `docs:`) make history scannable and toolable
- Branch naming: `feat/login`, `fix/empty-cart`, `42-login-redirect` — lowercase, hyphen-separated
- Small, frequent PRs beat giant ones — 100 lines gets reviewed, 3000 gets skimmed
- 15 cookbook scenarios cover 90% of real-world situations
- The simplest workflow you can keep is the best one

---

## 🧠 Detailed Notes

### 1. The Three Proven Workflows

#### Git Flow — branches for everything

| Branch | Role |
|--------|------|
| `main` | Always releasable — the only thing that ships |
| `develop` | Integration line where features accumulate |
| `feature/*` | New work, merged into `develop` |
| `release/*` | Pre-release stabilization, branched from `develop` |
| `hotfix/*` | Emergency fixes straight to `main` |

**Strength:** Discipline around releases and hotfixes. **Cost:** Ceremony. Best for versioned release trains with long maintenance branches.

#### GitHub Flow — the minimalist favourite

1. Anything in `main` is deployable
2. Branch per feature or fix — small and short
3. Open PR early, review continuously
4. Merge to `main`, deploy, repeat

No `develop`, no release branches. Fits teams that deploy continuously.

#### Trunk-Based Development — branchless small teams

One branch (`main`), everyone commits directly in small increments, protected by feature flags and CI. Maximizes integration frequency, minimizes merge pain — but demands disciplined, test-covered changes.

**Which to choose:**

| Context | Recommendation |
|---------|---------------|
| Small team, continuous delivery | GitHub Flow or trunk-based |
| Release trains with versioned releases | Git Flow |
| Rule of thumb | The simplest workflow you can keep |

Whatever you choose, agree on it, document it, let branch protection enforce it.

### 2. Commit Message Conventions

**Conventional Commits:**
```
feat: add login page
fix: handle empty cart on checkout
docs: explain setup steps
refactor: extract payment validator
chore: bump dependencies
test: cover the cart reducer
BREAKING CHANGE: change public API of auth
```

`git log --oneline` becomes a scannable changelog. Tools auto-generate release notes and bump versions from prefixes.

**Why consistent style pays off:** `git log --grep`, `git blame`, release notes all become trivial. Costs seconds per commit, pays for itself the first time you search `fix:` across a year of history.

**Branch naming conventions:** `feat/login`, `fix/empty-cart`, `docs/api`, or issue-driven `42-login-redirect`. Lowercase, hyphen-separated, meaningful — branch list reads like a roadmap.

### 3. Repository Housekeeping

**Files every repo deserves:**

| File | Purpose |
|------|---------|
| `README` | Front door — what, why, how to run. Rendered on repo page |
| `LICENSE` | Legally necessary for public code |
| `CONTRIBUTING` | How to set up, test, submit changes |
| `.gitignore` | Keep junk out |
| `.editorconfig` | Consistent formatting across editors |

**Templates:** `.github/ISSUE_TEMPLATE/` and `.github/PULL_REQUEST_TEMPLATE.md` force proper descriptions.

### 4. Code Review Culture

- **Small, frequent PRs** beat giant ones — split big work into mergeable chunks
- **Review for behavior, not style** — linters (Prettier, Black, gofmt) solve style
- **Automation replaces rules** — every rule enforced in CI is one less thing for humans to remember

### 5. Cookbook: 15 Everyday Scenarios

| # | Scenario | Solution |
|---|----------|----------|
| 1 | Committed to wrong branch | `git cherry-pick <hash>` on right branch, `git reset --hard HEAD~1` on wrong |
| 2 | Forgot a file | `git add file` + `git commit --amend --no-edit` |
| 3 | Committed a secret | Rotate immediately. Remove from history with `filter-repo`. Add to `.gitignore` |
| 4 | Deleted a file I need | `git checkout HEAD -- file` |
| 5 | Wrong author | `git commit --amend --author="Name <email>" --no-edit` |
| 6 | Same fix on two branches | `git cherry-pick <hash>` on each |
| 7 | Working tree is a mess | `git stash -u` parks everything; `git clean -fd` removes leftovers |
| 8 | Accidentally pushed to `main` | `git revert` — never force-push shared history |
| 9 | Local branch behind remote | `git pull --rebase` |
| 10 | Deleted branch by mistake | `git reflog`, find hash, `git branch recovered <hash>` |
| 11 | Merged too early | `git revert -m 1 <merge-hash>` — undo merge, keep main line |
| 12 | Split one commit into several | `git reset HEAD~1`, then `git add -p` for pieces |
| 13 | Copy file from another branch | `git restore --source feature -- app.py` |
| 14 | Undo a whole uncommitted day | `git stash push -u -m "before refactor"` — never reach for `--hard` first |
| 15 | Something is broken; who did it? | `git bisect start; git bisect bad; git bisect good <hash>; git bisect run npm test` |

> **Warning:** For scenario 3 (old secret), the accepted tool is `git filter-repo`. It rewrites history and forcibly updates every clone — a team operation, not a casual command.

### 6. Command Cheat Sheet

**Setup and repository:**
```bash
git init                                  # turn folder into repo
git clone <url>                           # copy repo (full history)
git config --global user.name "N"         # set identity
git remote -v                             # list remotes
```

**Snapshots and history:**
```bash
git status                                # state of working tree + index
git add <file>                            # stage a change
git commit -m "msg"                       # record snapshot
git diff                                  # unstaged changes
git diff --cached                         # staged changes
git log --oneline --graph --all           # readable history
git show <hash>                           # inspect one commit
```

**Branches and merging:**
```bash
git branch                                # list branches
git switch -c <name>                      # create + move to branch
git merge <branch>                        # integrate branch
git branch -d <name>                      # delete merged branch
git cherry-pick <hash>                    # copy commit to current branch
```

**Remotes and collaboration:**
```bash
git fetch                                 # download remote commits, change nothing
git pull --rebase                         # fetch + replay on top
git push -u origin <branch>              # publish + set upstream
git push --force-with-lease               # rewrite safely (own branch only)
git tag -a v1.0.0 -m "..."               # create annotated tag
```

**Undoing:**
```bash
git restore <file>                        # discard working-tree edits
git restore --staged <file>              # unstage
git reset --hard <hash>                   # move branch + discard changes
git revert <hash>                         # add commit that undoes another
git stash                                 # park changes
git reflog                                # safety net for lost commits
```

---

## 📝 Research Journey

- **Why:** This chapter turns individual skill into team skill — workflows, conventions, and the cookbook that covers real situations
- **Struggle:** The 15 cookbook scenarios were a lot to absorb — but they cover 90% of real problems
- **Aha moment:** Having a cheat sheet you can grep beats memorizing everything — now I know where to look
- **Career link:** Knowing Git workflows (Git Flow vs GitHub Flow) is essential for joining any engineering team; Conventional Commits are standard in professional repos

---

## 🔗 Related Topics

- [[git_github/09-pull-requests-code-review.md]] — previous: PRs
- [[git_github/10-workflows-cookbook.md]] — this is the final chapter
- [[books_summaries/agile_project_management/knowledge/version-control-devops-ci-cd.md]] — Agile perspective on version control
- [[ai_ml/04-xp-engineering-practices-for-ml.md]] — XP practices for ML

---

## 🤔 Open Questions

- [ ] When should a team adopt trunk-based development over GitHub Flow?
- [ ] How to adapt Conventional Commits for ML model versioning?

---
*Source: git-github-practical/chapters/ch10-workflows-cookbook.tex*

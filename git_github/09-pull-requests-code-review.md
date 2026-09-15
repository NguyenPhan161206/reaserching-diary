# Pull Requests and Code Review

**Created Date:** 2026-09-15
**Tags:** `#GitHub` `#PullRequest` `#CodeReview` `#BranchProtection` `#GitHubFlow`
**References:** [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow), [PR Best Practices](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- GitHub Flow: branch → commit → push → open PR → review → merge → repeat
- PR = conversation, not delivery — make it easy for reviewers
- **3 merge strategies:** merge commit (preserves all), squash (one clean commit), rebase (linear, every commit preserved)
- Branch protection = guardrails: require reviews + CI status checks
- `Closes #42` in PR description auto-closes issue on merge
- Keep PRs small — 100 lines gets reviewed properly; 3000 gets skimmed

---

## 🧠 Detailed Notes

### 1. GitHub Flow

```
Create a branch → Commit changes → Push branch → Open a PR
      ↑                                              ↓
   Merge ← Review & discuss ← ← ← ← ← ← ← ← ← ←
```

The whole workflow is a small cycle, repeated many times a day.

### 2. Opening a Pull Request

**Prerequisites:** Push your branch first: `git push -u origin feature/login`

**Web:** After push, GitHub shows yellow banner "Compare & pull request" → choose base (`main`) and compare (your branch) → write title + description → Create.

**CLI:**
```bash
gh pr create --fill            # title/body from commits
gh pr create --title "Add login" --body "Closes #12"
gh pr view --web               # open in browser
```

**GUI:** GitHub Desktop → push → "Create Pull Request" → opens browser.

**Writing PR descriptions:** Answer three questions the diff can't: **what** changed, **why**, and **how it was tested**. Link the issue (`Closes #42`). Include screenshots/demo URLs. Reviewers should never reverse-engineer your intent.

### 3. The PR Lifecycle

**Draft PRs:** Work in progress — run checks but can't be merged. Signal "please look but don't merge yet." Flip to "Ready for review" when done.

**Requesting reviewers:** Assign from right sidebar. `CODEOWNERS` files specify who must review specific paths.

**Adding commits:** Just push to the branch — PR tracks it. `git add` → `git commit` → `git push`. Reviewers see every new commit.

**Keeping current with upstream:** While PR is open, `main` moves on:
```bash
git switch feature/login
git pull --rebase origin main
```
Rebase keeps PR history readable. Resolve conflicts as usual, push again.

### 4. Reviewing: Comments, Suggestions, Approvals

**Inline comments:** Comment on specific lines. **Suggestion feature** embeds editable code blocks — "Add to batch" → "Commit suggestion" — turns review into collaborative edit.

**Review states:**

| State | Meaning |
|-------|---------|
| **Approve** | Good to merge |
| **Request changes** | Blocks merge until revised |
| **Comment** | General feedback, non-blocking |

**Required reviews:** With branch protection, PR needs N approvals before merge button enables. Combined with required status checks (CI must be green) = nothing reaches `main` unreviewed and untested.

**Etiquette:**
- Review for behavior and design, not style (linters automate style)
- Be specific: quote the line and explain **why**
- Say what is good, not only what is wrong
- Keep PRs small enough to actually review (few hundred lines max)

### 5. Three Ways to Merge

| Strategy | History | Best for |
|----------|---------|----------|
| **Create a merge commit** | Preserves every commit + merge node | Faithful history, but merge bubbles |
| **Squash and merge** | Folds branch into **one** commit on `main` | Messy WIP commits — most teams |
| **Rebase and merge** | Replays each commit onto `main`, no merge node | Curated, meaningful commits |

**Two sane defaults:**
- **Squash** when branch is one feature with messy commits
- **Rebase** when commits are themselves meaningful units

> **Tip:** Avoid many merge commits on `main`. Each is a discontinuity — `git log --first-parent` and blame become harder. Squash/rebase keep `main` readable.

### 6. Branch Protection Rules

**What they protect:** The main branch — without rules, anyone with write can push directly.

**Common rules:**
- Require pull request reviews (e.g., at least 1 approval)
- Require status checks to pass (CI must be green)
- Require linear history (forbids merge commits)
- Require signed commits
- Include administrators

**Setup:** `Settings → Branches → Add rule` for `main` → tick requirements → save.

### 7. Closing the Loop

- **Link issues:** `Closes #42` in PR description auto-closes issue on merge
- **Delete branch after merge:** GitHub offers button; CLI: `git branch -d` + `git push origin --delete`
- **Closed unmerged:** Discards nothing — branch and commits remain. Reopen later if needed

> **Tip:** A pull request is a conversation. Reviewers give their time — make it easy: small PRs, descriptive title, link to the issue.

---

## 📝 Research Journey

- **Why:** PRs are the heart of team collaboration — understanding the full lifecycle makes code review productive, not painful
- **Struggle:** Choosing between merge/squash/rebase felt arbitrary until I understood what each does to history
- **Aha moment:** Small PRs get real reviews; large PRs get rubber-stamped. Size is the single biggest factor in review quality
- **Career link:** Open-source contributions via PRs build public portfolio; code review skills are essential for ML team collaboration

---

## 🔗 Related Topics

- [[git_github/08-github-platform.md]] — previous: GitHub platform
- [[git_github/10-workflows-cookbook.md]] — next: workflows
- [[git_github/04-branching-and-merging.md]] — branch operations
- [[git_github/05-remotes-push-pull-fetch.md]] — push/pull

---

## 🤔 Open Questions

- [ ] How to set up CODEOWNERS for an ML project with separate data/model/api owners?
- [ ] When should a team enforce linear history vs allowing merge commits?

---
*Source: git-github-practical/chapters/ch09-pull-requests.tex*

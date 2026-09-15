# Working with GitHub

**Created Date:** 2026-09-15
**Tags:** `#GitHub` `#Platform` `#CLI` `#Desktop` `#Issues` `#Actions` `#Codespaces`
**References:** [GitHub Docs](https://docs.github.com), [GitHub CLI](https://cli.github.com)
**Status:** Published
**Difficulty:** Beginner

---

## 📌 Key Takeaways (TL;DR)

- GitHub is where Git becomes social — everything has a point-and-click twin
- **Enable 2FA today** — no excuse, highest-impact security step
- Use `gh repo create` to go from local folder to GitHub repo in one command
- **GitHub Desktop** = visual branch graph + checkbox staging + conflict buttons
- **GitHub CLI (`gh`)** = PR, issues, releases from the terminal
- **GitHub Actions** = CI/CD in the cloud — test on every push automatically

---

## 🧠 Detailed Notes

### 1. The GitHub Account

**Profile** (`github.com/username`): pinned repos, contribution graph, activity. In hiring context, it's your résumé.

**Two-factor authentication:** Enable today: `Settings → Password and authentication`. Use authenticator app or hardware key. GitHub increasingly **requires** 2FA.

**SSH keys and PATs:** Manage under `Settings → SSH and GPG keys` and `Developer settings → Personal access tokens`. Give tokens narrowest scope, set expiry.

**Notifications:** Bell icon controls email. Watch repos you follow, mute noisy conversations. Triage is a real productivity skill.

### 2. Touring the Web Interface

Repository tabs:
- **Code** — file browser, branch switcher, README
- **Issues, Pull requests** — collaboration inbox
- **Actions** — CI/CD runs
- **Projects** — planning boards
- **Releases, Packages, Security** — shipping + safeguarding
- **Settings** — collaborators, branch protection, visibility

**Search** is powerful: `repo:torvalds/linux author:tglx`, `is:pr is:open label:bug`, `language:python stars:>1000`.

### 3. Creating Repositories

**Web:** `+` (top right) → New repository → name, visibility, initialize with README.

**CLI:**
```bash
gh repo create my-project --public --source . --push
```
`--source .` uses current directory; `--push` uploads in same step. Fastest path from local to remote backup.

**Visibility:** Public = visible/searchable by everyone. Private = you + invited collaborators. Open-source = public; anything with secrets = private.

**Fork:** A copy under your account for contributing without write access: fork → clone your fork → branch → push → open PR upstream.

### 4. GitHub Desktop

**When GUI is better:** Visual branch graph, checkbox staging, conflict resolution with buttons. Ideal for visual learners and quick commits. Terminal remains faster for complex ops — they complement each other.

**Flow:** File → Clone → branch menu → left panel stages → "Commit to ..." → "Push origin".

**Conflicts:** Lists conflicted files, offers Choose yours / Choose theirs / Open in VS Code.

### 5. GitHub CLI (`gh`)

**Install + auth:**
```bash
gh auth login    # follow prompts, browser flow recommended
```

**Everyday commands:**
```bash
gh repo create my-repo --public --source . --push
gh pr create --fill          # open PR for current branch
gh pr view --web             # open in browser
gh issue list                # open issues
gh repo clone user/repo
gh release create v1.0.0 --title "v1.0.0" --notes "..."
```

Wraps REST/GraphQL APIs behind friendly verbs. Recommended command-line companion to web UI.

### 6. Issues and Projects

**Issues** are tracked tasks, bug reports, or feature requests. Each has a number (`#42`), title, description, comment thread. Reference from commits/PRs: `Closes #42`.

**Labels, milestones, assignees:**
- Labels: `bug`, `enhancement`, `good first issue`
- Milestones: group issues by target release
- Assignees: who owns it

**Issue templates:** Under `Settings → Issues → Templates` — convert vague complaints into actionable reports.

**Projects:** Kanban boards (To do / In progress / Done). Automatable — "move to Done when PR merges."

### 7. GitHub Actions (Gentle Intro)

**CI/CD in one paragraph:** CI runs checks (tests, linting, builds) on every change. CD deploys. GitHub Actions runs these in the cloud on events (push, PR, schedule).

**First workflow** (`.github/workflows/ci.yml`):
```yaml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: python -m pytest
```

Push this file and the Actions tab starts running it.

### 8. Codespaces

- Full dev environment in browser, configured by repo (`.devcontainer/`)
- Try a codebase without installing anything
- Same branch, same checkout — just Git on a cloud machine

> **Tip:** Terminal = source of truth (works everywhere); GitHub Desktop = lowers learning curve; web UI + `gh` = collaboration. You'll use all four in one day.

---

## 📝 Research Journey

- **Why:** GitHub is the social layer on top of Git — understanding its full feature set unlocks collaboration
- **Struggle:** The number of features (Issues, Projects, Actions, Codespaces) was overwhelming at first
- **Aha moment:** You don't need to learn everything at once — start with repos + PRs, add Actions later
- **Career link:** GitHub profile is a developer's portfolio; Actions is essential for ML pipeline CI/CD

---

## 🔗 Related Topics

- [[git_github/07-tags-and-releases.md]] — previous: tags
- [[git_github/09-pull-requests-code-review.md]] — next: PRs
- [[git_github/05-remotes-push-pull-fetch.md]] — remote operations
- [[git_github/10-workflows-cookbook.md]] — workflows and best practices

---

## 🤔 Open Questions

- [ ] How to set up GitHub Actions for a Python ML training pipeline?
- [ ] What are GitHub Codespaces limitations for heavy compute workloads?

---
*Source: git-github-practical/chapters/ch08-github-platform.tex*

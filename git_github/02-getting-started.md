# Getting Started: Install, Configure, Connect

**Created Date:** 2026-09-15
**Tags:** `#Git` `#GitHub` `#Setup` `#SSH` `#Configuration`
**References:** [Git Downloads](https://git-scm.com/download), [GitHub SSH Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
**Status:** Published
**Difficulty:** Beginner

---

## 📌 Key Takeaways (TL;DR)

- Install Git with default options; use VS Code as default editor on Windows
- Configure `user.name` and `user.email` globally — use the same email as your GitHub account
- **SSH keys** are preferred over HTTPS tokens — one setup, no password prompts
- `git init` creates a repo from scratch; `git clone` copies an existing one with full history
- First commit loop: `init` → `add` → `commit` → `log`

---

## 🧠 Detailed Notes

### 1. Installing Git

**Windows:** Download from [git-scm.com](https://git-scm.com/download/win). Key choices during setup:
- Default editor: "Use Visual Studio Code as Git's default editor"
- Line endings: keep "Checkout Windows-style, commit Unix-style" (`core.autocrlf=true`)
- Installs Git Bash (Unix-style terminal)

**macOS:** `brew install git` (Homebrew). Check: `which git`

**Linux:**
```bash
sudo apt install git        # Debian/Ubuntu
sudo dnf install git        # Fedora
sudo pacman -S git          # Arch
```

**Verify:** `git --version`

### 2. First-Time Configuration

Git stamps every commit with your identity. Do this once per machine:

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

**Important:** Use the **same email as your GitHub account** — GitHub links commits to your profile by email. For privacy, use `username@users.noreply.github.com`.

**Default editor:**
```bash
git config --global core.editor "code --wait"      # VS Code
git config --global core.editor "nano"             # terminal
```

**Three configuration scopes** (later overrides earlier):

| Scope | File | Purpose |
|-------|------|---------|
| `--system` | `/etc/gitconfig` | Every user on the machine |
| `--global` | `~/.gitconfig` | Your account |
| `--local` | `.git/config` | One repository |

**View config:** `git config --list --show-origin`

### 3. Connecting to GitHub

#### Option A: SSH Keys (Recommended)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
# accept defaults; optionally set a passphrase

cat ~/.ssh/id_ed25519.pub
# paste into GitHub → Settings → SSH and GPG keys → New SSH key

ssh -T git@github.com
# Hi yourname! You've successfully authenticated...
```

**Tip:** Generate a separate key per machine rather than copying the private key around. Deleting a machine's key from GitHub is a one-click security fix.

#### Option B: HTTPS with Personal Access Token

GitHub no longer accepts account passwords for Git. Generate a PAT at `GitHub → Settings → Developer settings → Personal access tokens`.

- **Windows/macOS:** Git Credential Manager stores the token automatically
- **Linux:** `git config --global credential.helper store` (saves in plain text — fine on personal machine)

### 4. Creating a Repository

**From scratch:**
```bash
mkdir my-project && cd my-project
git init
# Initialized empty Git repository in .../my-project/.git/
```

**Clone existing:**
```bash
git clone git@github.com:user/repo.git   # SSH
git clone https://github.com/user/repo.git   # HTTPS
cd repo
```

Cloning copies the **entire history**, not just files. You get a working tree, `.git` with full history, and a remote named `origin`.

### 5. Your First Commit

```bash
git config --global user.name "Ada Lovelace"
git config --global user.email "ada@example.com"

mkdir notes && cd notes
git init
printf 'hello\n' > hello.txt
git add hello.txt
git commit -m "chore: add hello.txt"
# [main (root-commit) 9fceb02] chore: add hello.txt
#  1 file changed, 1 insertion(+)

git log --oneline
# 9fceb02 chore: add hello.txt
```

---

## 📝 Research Journey

- **Why:** Wanted a clean, correct setup guide I can refer back to — the 10 minutes here prevent the most common frustrations later
- **Struggle:** SSH key setup felt intimidating at first; the concept of public/private key pairs wasn't intuitive
- **Aha moment:** Once SSH is set up, push/pull just work forever — it's a one-time investment
- **Career link:** Every professional developer environment uses Git + SSH; setting this up correctly is table stakes for AI Engineer roles

---

## 🔗 Related Topics

- [[git_github/01-version-control-concepts.md]] — previous: mental model
- [[git_github/03-core-daily-workflow.md]] — next: daily workflow
- [[git_github/08-github-platform.md]] — GitHub features deep dive

---

## 🤔 Open Questions

- [ ] How does Git Credential Manager store tokens on different OS keychains?
- [ ] What are the security trade-offs between SSH keys and PATs for teams?

---
*Source: git-github-practical/chapters/ch02-getting-started.tex*

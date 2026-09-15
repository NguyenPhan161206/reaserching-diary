# Tags and Releases

**Created Date:** 2026-09-15
**Tags:** `#Git` `#GitHub` `#Tags` `#Releases` `#SemVer` `#Versioning`
**References:** [Semantic Versioning](https://semver.org), [GitHub Releases Docs](https://docs.github.com/en/repositories/releasing-projects-on-github)
**Status:** Published
**Difficulty:** Beginner

---

## 📌 Key Takeaways (TL;DR)

- Tags mark specific points in history that deserve a name: "v1.0.0 shipped"
- **Annotated tags** are preferred — they carry authorship, date, message, and can be GPG-verified
- Tags are **not pushed automatically** — you must `git push origin v1.0.0` or `--tags`
- **SemVer:** MAJOR (breaking) . MINOR (features) . PATCH (fixes)
- A GitHub **Release** = tag + release notes + downloadable assets
- Never move or delete a published tag — cut a new version instead

---

## 🧠 Detailed Notes

### 1. Annotated vs Lightweight Tags

| Type | What it stores | Use case |
|------|---------------|----------|
| **Lightweight** | Bare pointer to a commit (name + hash only) | Scratch labels, temporary markers |
| **Annotated** | Full object: tagger name/email, date, message, optional GPG signature | Releases, anything meaningful |

> **Tip:** Prefer annotated tags for anything meaningful. They carry authorship and a message, and can be cryptographically verified.

### 2. Working with Tags

```bash
git tag -a v1.0.0 -m "First public release"    # annotated
git tag v1.0.0-rc.1                              # lightweight (scratch label)

git tag                    # list all tags
git tag -l "v1.*"          # pattern match
git show v1.0.0            # tag + the commit it points to
git describe --tags        # nearest tag to HEAD (used in build versioning)
```

**Tag a past commit:** `git tag -a v0.9.0 41bb2cd -m "pre-release checkpoint"`

**Delete local:** `git tag -d v1.0.0-rc.1`

### 3. Semantic Versioning (SemVer)

| Component | Meaning | Example |
|-----------|---------|---------|
| **MAJOR** | Incompatible API changes | `2.0.0` — requires migration |
| **MINOR** | Backward-compatible new features | `1.2.0` — drop-in upgrade |
| **PATCH** | Backward-compatible bug fixes | `1.0.1` — safe upgrade |

**Pre-release suffixes:** `1.0.0-alpha.1` → `1.0.0-rc.1` → `1.0.0`

**Why agree on SemVer:** Your tags become the public language of "what can I safely upgrade to." Dependency management, changelog generation, and support questions all get easier.

### 4. Pushing Tags to GitHub

Tags are **not pushed automatically** — they are not branches.

```bash
git push origin v1.0.0          # push one tag
git push origin --tags          # push all tags
```

**Delete remote tag:** `git push origin --delete v1.0.0-rc.1`

### 5. GitHub Releases

A **Release** = tag + presentation: release notes, attachments, and a permalink (`/releases/tag/v1.0.0`).

**From the web:** Navigate to Releases → Draft a new release → choose tag → fill notes.

**From CLI:**
```bash
gh release create v1.0.0 --title "v1.0.0" --notes "..." ./dist/*.tar.gz
```

**Release notes:** GitHub can auto-generate from merged PRs and closed issues. Hand-write a summary section — "Highlights", "Breaking changes", "Bug fixes" — for anything user-facing.

**Attaching assets:** Compiled binaries, language-specific packages, SBOM, SHA-256 checksums.

> **Warning:** Never move or delete a published tag — others may have built against it. If wrong, cut a new version instead.

---

## 📝 Research Journey

- **Why:** Tags and releases are the bridge between "code works" and "users can consume it"
- **Struggle:** Understanding when to bump MAJOR vs MINOR required thinking about API contracts from a user's perspective
- **Aha moment:** SemVer isn't just versioning — it's a communication contract with your users
- **Career link:** For ML model releases, semantic versioning helps track when model behavior changes — essential for A/B testing and rollback decisions

---

## 🔗 Related Topics

- [[git_github/06-undoing-and-recovery.md]] — previous: undoing
- [[git_github/08-github-platform.md]] — next: GitHub platform
- [[git_github/09-pull-requests-code-review.md]] — PR before release
- [[git_github/10-workflows-cookbook.md]] — release workflows

---

## 🤔 Open Questions

- [ ] How should ML model versions be tagged when performance varies by dataset?
- [ ] What's the relationship between SemVer and container image tagging?

---
*Source: git-github-practical/chapters/ch07-tags-releases.tex*

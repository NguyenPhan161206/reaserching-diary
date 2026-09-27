# Software Engineering Fundamentals: Agile, Release, and Git Tags

**Created Date:** 2026-09-12
**Tags:** `#SoftwareEngineering`, `#Agile`, `#Git`, `#CI_CD`
**Source:** Q&A with Agent

---

## 1. Agility in Software Development
**Concept & Mindset:**
- Not a rigid process, but a **mindset**: continuous iteration, breaking work into small pieces, delivering continuously, gathering feedback and adapting.
- **Core lesson:** welcome change, prioritise direct communication over process, and prefer a working product early (*fail fast, learn fast*) over secretly building a perfect project that then fails.

**Benefits for project management & AI:**
- Fixes Waterfall's main weakness (requirements are frozen up front while the market moves on).
- Splitting a project into sprints (1–4 weeks) reduces risk (risk mitigation).
- **In the AI era:** AI carries extreme uncertainty (there is no guarantee a model converges). Agile pushes you toward a modular architecture (separating data, training and inference) so components stay replaceable, which blocks the sunk-cost fallacy of grinding on a bad model, and it fits the data-collection loop (RLHF) perfectly.

## 2. Release & Release Build
**From a software engineering perspective:**
- **Release (verb — deployment):** the act of pushing code from a dev/staging branch straight into the production environment for users (i.e. deploy). Usually automated through a CI/CD system (GitHub Actions, Jenkins).
- **A release build (noun — artifact):** a snapshot of the whole source code at a sufficiently stable point in time. It is a pre-built package of source code (such as a `.jar`, `.apk` or Docker image) shipped with release notes (a changelog: new features, bug fixes).

## 3. Git Tags (Name Tags)
A "tag" is not a physical file — it is Git metadata (hyperlinked data) used to mark an important point in history (e.g. `v1.2.0`). Tags are immutable (read-only).

**Where to find tags:**
- **On the web interface (GitHub/GitLab):** in the "Releases" section on the right of a repository page, or in the branch dropdown (switch to the Tags tab).
- **In the terminal (local):** run `git tag` to list them, or `git show v1.2.0` for details. Git stores this data implicitly in the hidden `.git/refs/tags/` directory.
- **In the IDE (VS Code):** in the "Source Control" panel, under "TAGS" at the bottom.

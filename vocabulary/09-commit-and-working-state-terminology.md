# Vocabulary & Collocation — **COMMIT** & **UNCOMMITTED** (git, database, agile, business, psychology)

**Created Date:** 2026-09-27
**Last Updated:** 2026-09-27
**Tags:** `#vocabulary`, `#collocations`, `#word-family`, `#git`, `#database`, `#transactions`, `#agile`, `#business-english`, `#viet-glish`
**Status:** Active | Reviewed
**Template:** Template V — Advanced Vocabulary & Collocation
**Source note:** Vietnamese *commit / cam kết / thay đổi chưa commit*

---

## 📌 Term List & Quick Reference

| Term | IPA | Core meaning | Domain |
|---|---|---|---|
| **commit** (v) | `/kəˈmɪt/` | To bind yourself to a course of action; to carry out (a crime); to record permanently in a repository | Everyday / Legal / Git |
| **commit** (n) | `/ˈkɒm.ɪt/` | One recorded snapshot in a version history; a database transaction commit | Git / DB |
| **uncommitted** (adj) | `/ˌʌn.kəˈmɪt.ɪd/` | Not yet recorded / not yet made permanent | Git / DB / Social |
| **commitment** (n) | `/kəˈmɪt.mənt/` | An obligation undertaken; a pledged resource | Business / Agile / Social |
| **committed** (adj) | `/kəˈmɪt.ɪd/` | Bound to a course of action; fully dedicated | All |

> ⚠️ **Spelling:** **uncommitted** and **commitment** take a **double `t`** (*un-committed*, *commit-ment*). Frequent errors: ~~uncommited~~, ~~comitment~~, ~~commitmnet~~.

---

## 🧠 Detailed Vocabulary Analysis

### 1. The core idea of *commit*

Latin **committere** = *com-* ("together, wholly") + **mittere** ("to send") → **"to send wholly, to put into final form."** Every English sense is a consequence of that: once something is **committed**, it is **whole and final**.

| Sense | Meaning | Vietnamese | Example |
|---|---|---|---|
| **1. Bind oneself** | To pledge oneself to a course of action | cam kết, thủ định | *She committed to a career in AI engineering.* |
| **2. Carry out (something negative)** | To perform a crime, error or offence | phạm tội, mắc lỗi | *He **committed** fraud.* |
| **3. Record permanently (git)** | To write a snapshot into the repository history | commit, ghi vào lịch sử | *I **committed** the vocabulary notes and pushed.* |
| **4. Persist (data / storage)** | To make data permanent / final | commit, ghi cố định | *Flush the buffer to **commit** the data to disk.* |
| **5. Noun: one snapshot (git)** | A single recorded change | một bản commit | *Each **commit** should be atomic.* |
| **6. Noun: transaction commit (DB)** | The point at which a transaction becomes permanent | commit giao dịch | *The transaction will **commit** at the end of the unit of work.* |
| **7. Allocate resources (business)** | To earmark money, capacity or people | cam kết (nguồn lực) | *We have **committed** 30% of the budget to cloud spend.* |
| **8. Noun: pledge** | An obligation undertaken | cam kết, nghĩa vụ | *The bank issued a **commitment letter**.* |

#### ⚠️ The two critical collocations nobody teaches

1. **`commit to` + gerund, never `commit to` + bare verb.**
   ✅ *I committed to **studying** English daily.*
   ❌ *I committed to study English daily.*
   ✅ *She committed herself to learning Python.*
2. **`commit a crime`, never `do a crime`.**
   ✅ *commit a crime / an offence / an error / fraud / a sin*
   ❌ *do a crime* (a serious learner error)

*(`commit suicide` is also a fixed collocation — the verb is required, not optional.)*

### 2. PRONUNCIATION — the noun shifts stress

| Form | IPA | Note |
|---|---|---|
| **commit** (verb) | `/kəˈmɪt/` | stress on the **second** syllable |
| **commit** (noun) | `/ˈkɒm.ɪt/` | stress on the **first** syllable |
| **committed** | `/kəˈmɪt.ɪd/` | |
| **uncommitted** | `/ˌʌn.kəˈmɪt.ɪd/` | secondary stress on `un` |
| **commitment** | `/kəˈmɪt.mənt/` | |

This verb/noun stress shift catches learners who only ever hear the noun "a committee meeting" vs the verb "we commit".

### 3. Git vocabulary in one place

#### The state pipeline — where each word belongs
```
[Working tree]  dirty: modified files, untracked files
      │  git add          →  "uncommitted changes" that are STAGED
      │  git commit       →  uncommitted changes become a COMMIT (local history)
      │  git push         →  commit becomes SHARED (remote / origin)
      │  PR + merge       →  main branch → release
```

| Term | Vietnamese | Meaning |
|---|---|---|
| **working tree / working directory** | cây làm việc | The files as they exist on disk right now |
| **dirty** | có thay đổi chưa commit | Informal but standard: *"You have a dirty working tree."* |
| **clean** | sạch | *"nothing to commit, working tree clean"* |
| **staged** | đã `git add` | In the index, waiting to be committed |
| **unstaged** | chưa `git add` | Modified but not added |
| **uncommitted** | **chưa commit** | ⚠️ **Wider than *unstaged*** — staged items are also uncommitted |
| **untracked** | chưa được theo dõi | Brand-new file git has never seen |
| **commit** | một bản ghi | An immutable, content-addressed snapshot |
| **commit message** | nội dung commit | Conventionally imperative: `add:`, `fix:`, `docs:` |
| **amend** | sửa commit vừa rồi | Fix the last commit — ⚠️ never on shared history |
| **revert** | hoàn tác bằng commit mới | Safe; adds a new commit that undoes an old one |
| **reset** | di chuyển con trỏ HEAD | Destructive; discards commits |
| **HEAD** | con trỏ commit hiện tại | What `git log` starts from |
| **push / pull / fetch / clone** | đẩy / tải về / lấy dữ liệu / sao chép | |
| **origin** | remote mặc định | The upstream repository |
| **squash** | gộp nhiều commit thành một | Used in pull requests |
| **stash** | cất giữ thay đổi chưa commit | Lets you switch branch safely |
| **atomic commit** | commit nguyên tử | One logical change — the golden rule |

#### ⚠️ `uncommitted` vs `unstaged` vs `untracked` — the three-way distinction

| State | Question it answers | `git status` wording |
|---|---|---|
| **untracked** | Does git know this file exists? | `Untracked files:` |
| **unstaged** | Modified, but not yet `git add`ed? | `Changes not staged for commit:` |
| **uncommitted** | Not yet part of the history? | `Changes to be committed:` **and** `Changes not staged for commit:` |

- **uncommitted ⊃ unstaged.** A staged file is uncommitted but not unstaged.
- **Uncommitted ≠ unsaved.** Uncommitted means "not in git history"; a file can be fully saved on disk and still be uncommitted.
- **Uncommitted ≠ unpushed.** After `git commit` but before `git push`, you have local commits that are committed but unpushed.

### 4. Database sense — the one that is genuinely different

In databases, *uncommitted* refers to data that exists only in a volatile buffer, visible to some sessions and invisible to others.

| Isolation level | Dirty read? | Typical use |
|---|---|---|
| **Read uncommitted** | ✅ Yes | Never correct; **lowest** isolation level |
| **Read committed** | ❌ | The **default** in most databases (PostgreSQL, SQL Server, Oracle) |
| **Repeatable read** | ❌ | Snapshot stability within a transaction |
| **Serializable** | ❌ | Full serial ordering; highest isolation |

- The uncommitted data another transaction can see is called a **dirty read** — the reason `READ UNCOMMITTED` is almost never the right choice.
- **Uncommitted work** is also everyday developer language for work not yet persisted: *"Save your uncommitted work before the update."* Here it means **unsaved**, spanning editors, forms and databases.
- **Commit** here is the second half of **ACID**: *Atomicity* means the transaction either **commits** fully or rolls back entirely. `COMMIT` / `ROLLBACK` are literally SQL statements.

### 5. Agile sense — committed vs uncommitted sprint backlog

In Scrum, the Sprint Backlog is explicitly split:

| Item class | Meaning |
|---|---|
| **Committed** Sprint Backlog items | Work the team has committed to deliver this sprint |
| **Uncommitted** items | Extra work pulled in by the team — the *stretch* buffer |

- The Sprint Goal and the **commitment** are two different things; a team can be committed without promising everything.
- **Committed capacity** in capacity planning = capacity already promised to other work, so it is unavailable.
- The **Burndown** chart historically plotted ideal vs actual remaining committed work.
- "The team did not commit to that story" = it was never in the commitment.
- This is the same word as the finance meaning: a **commitment** is a promise that consumes future capacity.

### 6. Business & finance sense

| Term | Vietnamese | Meaning |
|---|---|---|
| **commitment letter** | thư cam kết | A bank's written promise to lend (mortgage pre-approval) |
| **firm order vs indicative order** | đơn hàng chắc chắn / dự kiến | Only the firm order is a commitment |
| **committed spend** | chi phí đã cam kết | Contracted, non-discretionary |
| **committed quantity** | số lượng đã cam kết | SAP/ERP: firm planned order vs forecast |
| **S&OP buckets: firm / committed / forecast** | chắc chắn / cam kết / dự báo | The classic sales-and-operations planning vocabulary |
| **take-or-pay** | trả hoặc không dùng | Minimum payment regardless of usage |
| **commit capital** | cam kết vốn | Invest capital that cannot be withdrawn freely |
| **non-cancellable commitment** | cam kết không thể hủy | The cost that cannot be recovered — compare [[economics_business/03-sunk-cost-fallacy.md]] |

### 7. Psychology — the most consequential sense

| Term | Meaning |
|---|---|
| **commitment** | A pledge that shapes behaviour — romantic, organisational or financial |
| **organisational commitment** | An employee's identification with an organisation (affective / normative / calculative) |
| **escalation of commitment** | Continuing to invest in a failing course of action **because** you have already invested |
| **commitment trap** | A decision that only looks good because of past investment |

> **Bridge to economics:** **escalation of commitment is the psychological mechanism behind the sunk cost fallacy.** A rational cost analysis says stop; the psychology says continue. Naming both in the same breath is what separates an engineer from a bystander. See [[economics_business/03-sunk-cost-fallacy.md]].

### 8. Word family

| POS | Form | Example |
|---|---|---|
| Verb | **commit** | *commit to studying* |
| Noun (git/DB) | **commit** | *an atomic commit* |
| Noun (pledge) | **commitment** | *a long-term commitment* |
| Adjective | **committed** | *fully committed to the project* |
| Adjective | **uncommitted** | *uncommitted changes* |
| Noun (archaic/legal) | **committal** | *the committal hearing* (UK: preliminary hearing) |
| Verb (British, non-standard) | *uncommit* | Colloquial: *"I can't uncommit now."* — **not** a git command |

### 9. ⚖️ Confusables Comparison Matrix

| Word | Core meaning | Typical context | Example |
|---|---|---|---|
| **commit** | Bind yourself; make final | Contract, git | *I committed to the deadline.* |
| **submit** | Hand something over; it may be refused | Forms, tenders, papers | *I submitted the application.* |
| **commission** | Authorise and pay for; a fee; a body | Business, legal | *They **commissioned** a feasibility study.* / *a 5% **commission**.* / *a royal **commission**.* |
| **entrust** | Give someone responsibility for | Management | *She entrusted him with the keys.* |
| **concentrate** | Focus | Everyday | *I can't concentrate.* |
| **committee** | A group of people | Organisations | *the steering committee* — ⚠️ **not** "a commit" |
| **commitment (psych.)** | A pledge that drives behaviour | Relationships, orgs | *He showed real commitment.* |
| **uncommitted (DB)** | Not yet permanent, possibly dirty | Databases | *dirty read of uncommitted data* |

> **⚠️ `commission` vs `commit`** is the classic learner collision: *commission a report* = **ủy thác/bàn dịch** a report; *commit a report* is **wrong**. Also `committee` (a group) is a completely different word from `commit`.

### 10. Viet-Glish pitfalls

| ❌ Viet-Glish / common error | ✅ Native standard | Why |
|---|---|---|
| "I commit to study English every day." | "I **committed to studying** English every day." | `to` + gerund after *commit to* |
| "I commit myself to do that." | "I **committed myself to doing** that." | Same rule |
| "He did a crime." | "He **committed** a crime." | Fixed collocation |
| "**uncommited** changes" | "**uncommitted** changes" | Double `t` |
| "I did a commit yesterday." | "I **made** a commit yesterday." | Noun *commit* takes *make* |
| "I committed a small mistake." | "I **made** a small mistake." (or *committed an error* — formal) | *commit a mistake* is technically possible but very rare; *made a mistake* is natural |
| "commit file / commit the change" | "**commit the file** / **commit the changes**" | Needs the determiner; `changes` is usually plural |
| "Thay đổi uncommitted của tôi bị mất." | ✅ correct, but note: after `git add` they are still uncommitted — the real risk is *uncommitted **and** unstaged*** | Precision matters for backup advice |
| "Công ty commit 30% ngân sách cho cloud." | ✅ correct — *committed spend*, *we have committed 30% of the budget* | Business register is right here |
| "Anh ấy rất uncommitted trong mối quan hệ." | ✅ colloquial, means *chưa cam kết, còn đang suy nghĩ* | Social sense, not a fault in English |
| "Tôi không thể uncommit." | ✅ colloquial British usage is fine, but in git the correct verb is **`git reset`** or **`git revert`** | Precision about the tool |
| "commit" as a transitive verb alone | ❌ "he committed" is ambiguous between *phạm tội* and *git commit* | Always give the object |

### 11. Cross-Domain Bridge (Tech ↔ Business ↔ Social Science)

**Git & distributed systems**
- A commit is **content-addressed and immutable**: its hash covers the tree, the parents, the author and the message. That is why amending shared history is dangerous — it silently rewrites an identifier other people already have. This is the mechanical reason behind the rule *"never force-push a shared branch."*
- **Atomic commits** are the unit of review. A commit that changes behaviour and formatting at once cannot be reviewed, bisected or reverted cleanly.
- **Signed commits** (GPG / SSH) turn a commit into an **attested** event — the same primitive behind signed releases and supply-chain provenance.

**Databases & distributed transactions**
- `COMMIT` / `ROLLBACK` implement **atomicity**; the **two-phase commit (2PC)** coordinator protocol is how a distributed transaction commits across services — and how it can get **stuck** in doubt if a participant fails.
- **Read uncommitted → dirty read** is the canonical example of a performance optimisation that creates a correctness bug, and it is a guaranteed interview question.

**Agile & project management**
- **Committed vs uncommitted backlog** and **committed capacity** are the vocabulary of delivery promises. This links directly to the existing Agile notes in `books_summaries/05-agile_project_management/`.
- The everyday question *"what have we committed to?"* is a scope question, and the honest answer is read off the commitment, not off the wish list.

**Business & economics**
- **Firm / committed / forecast** buckets in S&OP decide what the supply chain may physically build. Misreading *committed* as *forecast* is expensive.
- **Commitment devices** in contract theory lock in a party who would otherwise renegotiate — the financial twin of escalation of commitment.
- **Take-or-pay** and non-cancellable commitments are why cloud cost management is a commitment problem, not a usage problem: *"Our AWS bill is a commitment, not a variable."*

**Security**
- **Attestation and commit signing** answer *"who authorised this change?"* — a question an auditor will ask about any banking system, and one your project plans should already answer.

**Psychology & society**
- **Escalation of commitment** explains sunk costs, repeated failed migrations and stubborn architectures. It is also why *commitment* appears in organisational behaviour research on why good people stay in failing teams.

---

## 🧪 Practice Section

### A. Choose the correct word
1. She ______ herself to learning Japanese before the trip. *(commit / commission / committee)*
2. The database will ______ the transaction at the end of the unit of work. *(commit / commit to / commit up)*
3. I made a small ______ but it is already pushed, so I cannot amend it. *(commit / commitment / committee)*
4. `git status` reports five **uncommitted** files, two of which are not yet staged. → what does "uncommitted" include here? *(only unstaged files / both staged and unstaged files / untracked files only)*

### B. Fill in the collocations
5. Never ______ a crime — say "commit a crime". → *(complete the fixed collocation)*
6. The bank issued a letter of ______ before the mortgage. → *(complete)*

### C. Rewrite the Viet-Glish
7. "I commit to study AI every night." → ______
8. "uncommited changes" → ______
9. "They did a serious crime." → ______

### Answer Key
1. **committed** · 2. **commit** · 3. **commit** · 4. **both staged and unstaged files**
5. *do* · 6. **commitment**
7. *"I **committed to studying** AI every night."*
8. *"**uncommitted** changes"*
9. *"They **committed** a serious crime."*

---

## 🔬 Research Journey

- **Why I'm researching this:** *commit* looks like a git command, but the same word covers a romantic pledge, a crime, a database transaction, a sprint promise and a bank loan. Reading "uncommitted changes" while also reading "uncommitted transaction" in a SQL log is not a vocabulary curiosity — it is a genuine comprehension risk.
- **What I struggled with:** (a) `commit to` + gerund — I kept writing `to do`; (b) the verb/noun stress shift; (c) not realising that **uncommitted ⊃ unstaged**, which matters when telling someone their work is safe or at risk.
- **Aha moments:** (1) the Latin *mittere* = "to send" explains why *commit* means "put into final form" in both git and SQL; (2) **escalation of commitment is the psychology behind the sunk cost fallacy** — one sentence links my AI notes to my economics notes; (3) the Scrum distinction between *committed* and *uncommitted* backlog is the same accounting distinction as *firm* versus *forecast* in S&OP.
- **Career connection:** commits are the unit of code review, transactions are the unit of data integrity, and commitments are the unit of delivery promises. In a bank system, saying the wrong one in a meeting changes what gets built.

---

## 🔗 Related Topics

- [[vocabulary/08-property-multi-sense-lexicon.md]] — *property* layers its senses the same way; a model worth copying
- [[vocabulary/06-document-and-communication-verb-family.md]] — *state* versus *commit*: binding language in contracts
- [[economics_business/03-sunk-cost-fallacy.md]] — escalation of commitment as the psychological mechanism
- [[git_github/]] — version-control notes that use this vocabulary
- [[books_summaries/05-agile_project_management/]] — committed versus uncommitted sprint backlog
- [[software_architecture/01-event-driven-architecture-task-orchestration.md]] — where transaction boundaries and idempotency live
- [SUGGESTED] **Database transaction vocabulary** (`atomicity`, `isolation`, `dirty read`, `phantom read`, `deadlock`, `two-phase commit`, `rollback`, `idempotency`) — *why:* the terms `commit` and `uncommitted` are meaningless without them, and every backend interview and bug report uses them. → fits `vocabulary/`, bridge to `software_architecture/`
- [SUGGESTED] **Decision & commitment vocabulary** (`commitment device`, `sunk cost`, `opportunity cost`, `irreversibility`, `option value`, `real options`) — *why:* turns the behavioural term into an economic one, which is exactly the bridge this repository is built for. → fits `economics_business/`
- [SUGGESTED] **Code review & commit hygiene** (`atomic commit`, `conventional commit`, `changelog`, `semver`, `signed commits`, `bisect`) — *why:* professional habits that are cheap to build and immediately visible to any hiring team. → fits `git_github/`

---
*Precision in vocabulary reflects precision in engineering.*

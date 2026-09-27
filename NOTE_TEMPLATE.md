# Note Templates — Researching Diary

> 7 template variants cho từng domain. Agent PHẢI chọn đúng template khi tạo note mới.
> Xem AGENTS.md Section C để biết mapping folder → template.

---

## TEMPLATE A: Technology Note
**Sử dụng cho:** `ai_ml/`, `iot_aiot/`, và any tech-heavy content

```markdown
# [Technical Topic Title]

**Created Date:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Tags:** `#tag1`, `#tag2`
**References:** [Papers, docs, links]
**Status:** Draft | Review | Published
**Difficulty:** Beginner | Intermediate | Advanced

---

## 📌 Key Takeaways (TL;DR)
- Key technical insight 1
- Key technical insight 2

## 🧠 Detailed Notes

### 1. Core Concepts
- Explanation of fundamental concepts
- Why this matters (context)
- Visual/diagram description (if applicable)

### 2. How It Works (Mechanism)
- Step-by-step technical breakdown
- Math formulas (LaTeX): $formula$
- Data flow / architecture

### 3. Implementation (Code)
```language
// Code example with comments
// Must be runnable or clear pseudocode
```

### 4. Practical Application
- Real-world use cases
- Performance benchmarks (if available)
- Trade-offs and alternatives

### 5. Comparison Table
| Feature | Option A | Option B |
|---------|----------|----------|
| ... | ... | ... |

## 📝 Research Journey
- **Why:** [Personal motivation for researching this]
- **Struggle:** [What was conceptually difficult]
- **Aha moment:** [The breakthrough in understanding]
- **Career link:** [How this connects to AI Engineer goal]

## 🔗 Related Topics
- [[folder/file.md]] — connection explanation
- [SUGGESTED] topic-name — why it should be added

## 🤔 Open Questions
- [ ] Deeper question 1
- [ ] Deeper question 2

---
*Feynman Technique: Explain to a 12-year-old.*
```

---

## TEMPLATE B: Business/Economics Analysis
**Sử dụng cho:** `economics_business/`

```markdown
# [Business/Economics Topic]

**Created Date:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Tags:** `#tag1`, `#tag2`
**References:** [Books, papers, case studies]
**Status:** Draft | Review | Published
**Industry Focus:** [e.g., Tech, Finance, Healthcare]

---

## 📌 Key Takeaways (TL;DR)
- Business insight 1
- Business insight 2

## 🧠 Detailed Analysis

### 1. Theoretical Framework
- Core theory and originator(s)
- Key assumptions
- How it differs from competing theories

### 2. Market Evidence & Data
- Statistics and data points
- Historical performance
- Current market trends (2024-2026)

### 3. Real-World Case Studies
#### Case 1: [Company/Industry]
- Situation → Action → Result
- Key metrics (% growth, revenue, market share)
- What made it work/fail

#### Case 2: [Company/Industry]
- (Same structure)

### 4. Business Model Implications
- How this affects revenue models
- Cost structure considerations
- Competitive advantage opportunities

### 5. Actionable Strategies
- Strategy 1: [What to do + how + expected outcome]
- Strategy 2: [What to do + how + expected outcome]

## 📝 Research Journey
- **Why:** [Personal motivation]
- **Struggle:** [What was hard to understand]
- **Aha moment:** [Breakthrough]
- **Career link:** [AI Engineer + Economics intersection]

## 🔗 Related Topics
- [[economics_business/file.md]] — connection
- [[ai_ml/file.md]] — tech connection
- [SUGGESTED] missing-topic — why needed

## 🤔 Open Questions
- [ ] Market question
- [ ] Strategy question

---
*Economics explains human behavior. Business applies it.*
```

---

## TEMPLATE G: Dev Tool & Command Note
**Sử dụng cho:** `dev_foundation/linux/`, `dev_foundation/docker/`, `dev_foundation/yaml_config/`, `dev_foundation/web_basics/` (JS parts), `dev_foundation/databases/`
**Mục đích:** Học *qua thực hành lệnh* — mental model → cheat sheet → lab → pitfalls → practice. Tối ưu cho tra cứu nhanh khi đang code/debug.
**Khác Template A:** Template A giải thích *một hệ thống*; Template G giải thích *một công cụ, dùng tay*.

```markdown
# [Tool / Command Topic]

**Created Date:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Tags:** `#Linux`, `#Shell`, `#Filesystem` (chọn tag theo domain)
**References:** [Official docs, man pages, links]
**Status:** Draft | Review | Published
**Difficulty:** Beginner | Intermediate | Advanced
**Prerequisites:** [DSA / Linux / Git / etc.]

---

## 📌 Key Takeaways (TL;DR)
- 1 dòng quan trọng nhất về mental model
- 1 lệnh/pattern hay dùng nhất
- 1 gotcha đã cắm vào người

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)
> Viết cách *nghĩ* về hệ thống TRƯỚC khi dùng lệnh. Ví dụ: "Linux không có 1 cây thư mục, nó có 1 namespace hệ thống tệp từ `/`").
- Giải thích khái niệm nền
- Vì sao mental model này lại quan trọng (giải thích hành vi lệnh bất ngờ)

### 2. Mechanism (How It Works)
- Bước 1 → bước 2 → bước 3 (data flow / system call / config resolution)
- Có diagram ASCII nếu giúp hiểu
- Nêu rõ cái gì *thực sự* xảy ra dưới lệnh

### 3. Command Cheat Sheet

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `ls -lah` | ... | ... | `total 24K ...` |

> **Rule:** mỗi lệnh PHẢI có cột "When to use". Lệnh không có ngữ cảnh dùng là lệnh không học được.

### 4. Practical Lab

> Bài tập có đầu vào rõ + expected output để tự verify (không cần AI).

**Lab 1 — [Tên]**
```bash
# Step 1: ...
# Step 2: ...
# Expected output:
# ...
```

### 5. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| `sudo rm -rf ./` hiểu sai đường dẫn | ... | ... |

### 6. Comparison Table *(nếu applicable)*
| Feature | Option A | Option B |
|---------|----------|----------|
| ... | ... | ... |

## 🔗 Tech Cross-References
> **BẮT BUỘC** (AGENTS.md Section K): 2–4 công nghệ khác, mỗi cái 1–3 dòng ở tầng "vừa phải, dễ hiểu".
- **[Tech X]** — mối liên hệ 1 câu. [→ note trong repo]
- **[Tech Y]** — mối liên hệ 1 câu. [→ note trong repo]

## 🧪 Practice Tasks
> KHÔNG kèm lời giải. Làm xong nhờ AI chấm/chữa (biến thể: hard hơn, tối ưu, hỏi lý thuyết).
- [ ] Task 1 — mục tiêu, gợi ý command, điều kiện hoàn thành
- [ ] Task 2 — khó hơn Task 1 một bậc

## 📝 Research Journey
- **Why:** [Động cơ cá nhân]
- **Struggle:** [Chỗ khó hiểu nhất]
- **Aha moment:** [Khoảnh khắc "à, hóa ra..."]
- **Career link:** [Liên hệ mục tiêu AI Engineer]

## 🔗 Related Topics
- [[dev_foundation/...]] — connection explanation
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there (AGENTS.md Section L)
- [SUGGESTED] topic-name — why it should be added

## 🤔 Open Questions
- [ ] Deeper question 1
- [ ] Deeper question 2

---
*Type the commands. Break something. Read the error. That is the only real way to learn a tool.*
```

---

## TEMPLATE F: IELTS Listening Practice
**Sử dụng cho:** `ielts_training/listening/`

```markdown
# IELTS Listening Practice — [Source Title]

**Created Date:** YYYY-MM-DD
**Source:** [YouTube/Podcast/Article title + link]
**Duration:** XX:XX
**Tags:** `#IELTS`, `#Listening`, `#TechTopic` / `#BusinessTopic`
**Difficulty:** Progressive (Easy → Hard across N parts)
**Status:** Draft | Review | Complete

---

## 🎧 Source Overview

| Chapter | Timestamp | Topic |
|---------|-----------|-------|
| 1 | 0:00–X:XX | Topic summary |
| ... | ... | ... |

---

## PART 1 — [Exercise Type] (Easy)

> **Skill tested:** [e.g., Catching specific details]
> **Instructions:** [Standard IELTS instructions]
> **Source chapters:** X:XX–Y:YY

### Questions 1–N

[Questions in IELTS format]

---

### ✅ Answer Key

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **answer** | Source quote or explanation |

---

### 📝 Vocabulary — Part N

| Word/Phrase | Meaning | Example from Transcript |
|-------------|---------|------------------------|
| **term** | definition | "quote" |

---
---

## PART N — [Next exercise type] (Harder)

[Same structure as above]

---
---

## 📊 Self-Assessment

| Part | Questions | Your Score | Band Estimate |
|------|-----------|------------|---------------|
| Part 1 | N | __/N | |
| **Total** | **X** | __/X | |

### Band Score Guide (Listening)
| Score | Band |
|-------|------|
| 23–26 | 8.0–9.0 |
| 20–22 | 7.0–7.5 |
| 16–19 | 5.5–6.5 |
| 12–15 | 4.0–5.0 |
| < 12 | Below 4.0 |

---

## 📝 Research Journey
- **Why:** [Personal motivation for this practice]
- **Struggle:** [What was hard about this listening]
- **Aha moment:** [Breakthrough in understanding]
- **Career link:** [Connection to AI Engineer goal]

---

## 🔗 Related Topics
- [[other-note.md]] — connection

---
*Growth is measured, not assumed.*
```

---

## TEMPLATE C: Social Science / Psychology
**Sử dụng cho:** `psychology_society/`

```markdown
# [Social Science/Psychology Topic]

**Created Date:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Tags:** `#tag1`, `#tag2`
**References:** [Academic papers, studies, books]
**Status:** Draft | Review | Published
**Sub-field:** [e.g., Cognitive Psychology, Behavioral Economics, Political Science]

---

## 📌 Key Takeaways (TL;DR)
- Behavioral/psychological insight 1
- Behavioral/psychological insight 2

## 🧠 Detailed Analysis

### 1. Theoretical Framework
- Core theory and originator(s)
- Key assumptions and axioms
- Historical context of the theory
- How it differs from competing schools of thought

### 2. Empirical Evidence
#### Landmark Studies
- **Study 1:** [Author, Year]
  - Design: [experiment/quasi-experiment/longitudinal]
  - Sample: [N=, demographics]
  - Findings: [specific results with effect sizes]
  - Replication status: [replicated/contested/failed]

- **Study 2:** [Author, Year]
  - (Same structure)

#### Meta-analyses & Reviews
- Overall effect size across studies
- Heterogeneity and moderators
- Publication bias assessment

### 3. Real-World Applications
#### Application 1: [Context]
- How the theory manifests in practice
- Measurable outcomes (% change, effect size)
- Why it works (mechanism)

#### Application 2: [Context]
- (Same structure)

### 4. Critical Analysis
- **Strengths:** What this theory explains well
- **Limitations:** Where it falls short
- **Counter-arguments:** What critics say
- **Boundary conditions:** When does this NOT apply?

### 5. Ethical Implications
- Potential for exploitation/manipulation
- Power dynamics concerns
- Privacy considerations
- AI/tech intersection: How this knowledge could be misused in algorithms
- Responsible application guidelines

### 6. Connection to AI/Technology
- How this psychological principle appears in:
  - Recommendation algorithms
  - User experience design
  - AI bias and fairness
  - Human-AI interaction

## 📝 Research Journey
- **Why:** [Personal motivation]
- **Struggle:** [Conceptual difficulties]
- **Aha moment:** [Breakthrough]
- **Career link:** [How psychology informs AI Engineering]

## 🔗 Related Topics
- [[psychology_society/file.md]] — psychology connection
- [[economics_business/file.md]] — behavioral economics link
- [[ai_ml/file.md]] — AI application link
- [SUGGESTED] topic — why it's needed

## 🤔 Open Questions
- [ ] Research question requiring further study
- [ ] Contradiction noticed but unresolved
- [ ] How to apply this to AI system design

---
*Psychology explains WHY people behave as they do.*
*Understanding this makes you a better engineer, not just a better psychologist.*
```

---

## TEMPLATE D: Book Summary
**Sử dụng cho:** `books_summaries/`

```markdown
# [Book Title] — [Author]

**Created Date:** YYYY-MM-DD
**Tags:** `#tag1`, `#tag2`
**Rating:** ⭐⭐⭐⭐⭐ (1-5)
**Genre:** [e.g., Psychology, Business, Science]
**Status:** Summary | Analysis | Deep Dive

---

## 📌 Key Takeaways (TL;DR)
- Core message 1
- Core message 2
- Core message 3

## 📖 Book Overview
- **Author background:** [Who is this person?]
- **Publication year:** [Year]
- **Core thesis:** [One sentence summary]
- **Target audience:** [Who should read this?]

## 🧠 Detailed Analysis

### 1. Core Concepts (Chapters 1-5)
#### Concept 1: [Name]
- What the author argues
- Evidence/examples provided
- My analysis: [Agree/Disagree + Why]

#### Concept 2: [Name]
- (Same structure)

### 2. Profound Examples
- Example 1: [Story/case from the book]
  - Why it's powerful
  - How it challenges conventional thinking
- Example 2: [Story/case from the book]
  - (Same structure)

### 3. Cross-Book Connections
- How this connects to [other book 1]
- How this connects to [other book 2]
- Where the author disagrees with [other author]

### 4. How to Apply
- Personal action item 1
- Personal action item 2
- Career application (AI Engineer context)

## 📝 Research Journey
- **Why I read this:** [Motivation]
- **Biggest surprise:** [What I didn't expect]
- **Changed my mind about:** [Shift in thinking]
- **Will implement:** [Concrete action]

## 🔗 Related Topics
- [[books_summaries/other-book.md]] — connection
- [[psychology_society/file.md]] — if psychology-related
- [SUGGESTED] complementary-book — why to read next

## 🤔 Discussion Questions
- [ ] Question for further reflection
- [ ] Question to discuss with others

---
*Books are seeds. Notes are the garden.*
```

---

## TEMPLATE E: Evaluation Log
**Sử dụng cho:** `personal_evaluations/`

```markdown
# [Evaluation Topic]

**Created Date:** YYYY-MM-DD
**Purpose:** [Self-assessment, skill tracking, English eval]
**Session:** [Session # or date range]

---

## 📌 Summary
- Overall progress assessment
- Key areas of improvement
- Areas needing work

## 📊 Assessment Details

### Session 1: [Topic/Date]
- **Questions/Topics covered:**
  1. [Q]: [User's answer summary]
     - Strengths: ...
     - Areas to improve: ...
     - Corrected version: ...

- **Vocabulary/Key Terms:**
  - term1: definition
  - term2: definition

### Session 2: [Topic/Date]
- (Same structure)

## 📈 Progress Over Time
| Metric | Session 1 | Session 2 | Trend |
|--------|-----------|-----------|-------|
| Accuracy | X% | Y% | ↑/↓/→ |
| Vocabulary | N terms | M terms | ↑/↓/→ |
| Confidence | Low/Med/High | Low/Med/High | ↑/↓/→ |

## 🎯 Action Items
- [ ] Specific improvement 1
- [ ] Specific improvement 2

---
*Growth is measured, not assumed.*
```

---

## TEMPLATE V: Advanced Vocabulary & Collocation Note
**Sử dụng cho:** `vocabulary/`

```markdown
# Vocabulary & Collocation — [Topic / Term Group]

**Created Date:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Tags:** `#vocabulary`, `#collocations`, `#technical-english`
**Status:** Active | Reviewed

---

## 📌 Term List & Quick Reference

| Term / Phrase | IPA | Core Meaning | Domain |
|---------------|-----|--------------|--------|
| **term** | `/ipa/` | definition | AI / Business / System |

---

## 🧠 Detailed Vocabulary Analysis

### 1. [Term Name]
- **IPA:** `/.../`
- **Definition & Domain Context:** Precise technical/business explanation.
- **Word Family:** 
  - Noun: ...
  - Verb: ...
  - Adjective: ...
  - Adverb: ...
- **Collocations & Native Pairings:**
  - `verb + term` (e.g., ...)
  - `adjective + term` (e.g., ...)
  - `term + noun` (e.g., ...)
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "..."
  - ✅ *Native standard:* "..."
- **Resource & Literature Mapping** *(Conditional: Applied when term represents a technical problem or market/business pain point)*:
  - **Technical Architecture / Problem:** ...
  - **Academic Paper / Documentation:** ...
  - **GitHub Repo / Source Code Reference:** ...
- **Cross-Domain Bridge (Tech ↔ Economics):** How this concept links software engineering with business/economics.

## 📝 Research Journey
- **Why:** [Personal motivation for mastering these terms]
- **Struggle:** [Confusing nuances or Viet-Glish habits overcome]
- **Aha moment:** [Realizing the correct collocation or native pairing]
- **Career link:** [How precise technical English impacts AI Engineer career and technical communication]

## 🔗 Related Topics
- [[folder/file.md]] — technical context connection
- [SUGGESTED] related-vocabulary-topic — why to study next
```

---

## Template Selection Guide

```
User asks about topic
        │
        ▼
  ┌─────────────┐
  │ What folder? │
  └──────┬──────┘
         │
    ┌────┴────┬──────────┬──────────────┬──────────────┬────────────────┐
    ▼         ▼          ▼              ▼              ▼                ▼
  ai_ml/   economics  psychology   books/         personal/      dev_foundation/
  iot_aiot/ _business/ _society/   summaries/     evaluations/     linux/ docker/
    │         │          │              │              │            yaml_config/ web_basics/
    │         │          │              │              │            databases/ dsa/
    ▼         ▼          ▼              ▼              ▼                ▼
Template A Template B Template C  Template D  Template E      A / G / anchor
  (Tech)   (Business)  (Social)   (Book)      (Eval)     (G = tool/command)
```

**Rule nhớ nhanh:**
- Cần **dùng tay** một công cụ (lệnh shell, docker CLI, SQL, YAML) → **Template G**
- Cần **hiểu cơ chế hệ thống/kiến trúc** (layout engine, orchestration, schema design) → **Template A**
- Folder nào trộn cả hai → chia theo mục đích từng note, ghi rõ trong README của folder

---

*Last updated: 2026-09-28 — Config version: 3.3 (added TEMPLATE G — Dev Tool & Command Note)*

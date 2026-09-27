---
name: content-manager
description: Use when managing research diary content — creating, formatting, reviewing, organizing notes. Triggered by keywords: note, research, diary, template, organize, review, suggest, push, log, folder, new-note, contribution.
---

# Content Manager Skill

## When to Use
- User says "thêm note", "ghi nhận", "research", "ghi lại", "tạo note"
- User asks to review, format, or organize notes
- User asks to suggest improvements or missing topics
- User asks about push readiness
- User asks to propose new folders
- User says "log", "ghi lại", "đóng góp"

## Config Hierarchy
1. **AGENTS.md** (root) — PRIMARY rules. ALWAYS read this first.
2. **NOTE_TEMPLATE.md** — Template variants per domain.
3. **This file** — Workflow guidance only.

## Workflow: Create Note
```
1. Determine topic + folder
2. Select template variant (AGENTS.md Section C)
3. Write content (AGENTS.md Section A2 — depth standard)
4. Add Research Journey (AGENTS.md Section A4)
5. Add cross-references (AGENTS.md A5 — cross-folder links)
6. Add 🔗 Tech Cross-References if note is in dev_foundation/ (AGENTS.md Section K)
7. Add 🔗 cross-repo link if topic relates to another repo (AGENTS.md Section L)
8. Run depth checklist (AGENTS.md Section H)
9. Suggest related topics (AGENTS.md Section A3)
10. Update tracking files (AGENTS.md Section I + RESEARCH_AREAS.md)
11. Add entry to the folder README's "Note Index" table (AGENTS.md Section M)
12. Check push conditions (AGENTS.md Section G)
```

## Workflow: Review Notes
```
1. Select scope (folder or all)
2. Check template compliance
3. Check depth checklist
4. Check missing sections
5. Report by severity (🔴/🟡/🟢)
6. Fix on user confirmation
7. Log review actions
```

## Workflow: Push
```
1. Run git status + diff
2. Verify all conditions met
3. Display summary with commit messages
4. Ask confirmation
5. Execute push on confirm
6. Log push event
```

## Domain Detection
| Keyword in topic | Template | Folder |
|-----------------|----------|--------|
| AI, ML, model, neural, transformer, LLM | A (Tech) | ai_ml/ |
| IoT, MQTT, sensor, ESP32, STM32 | A (Tech) | iot_aiot/ |
| business, market, economy, strategy, revenue | B (Business) | economics_business/ |
| psychology, behavior, cognitive, bias, society | C (Social) | psychology_society/ |
| book, author, chapter, summary, read | D (Book) | books_summaries/ |
| evaluate, assessment, skill, English, IELTS | E (Eval) | personal_evaluations/ |
| idea, random, concept, thought | Flexible | random_ideas/ |
| git, branch, commit, PR, GitHub Actions | A (Tech) | git_github/ |
| vocabulary, collocation, word family, Viet-Glish | V (Vocabulary) | vocabulary/ |
| **shell command, terminal, chmod, filesystem, permission, pipe, grep, sed, process, systemd** | **G (Dev Tool)** | **dev_foundation/linux/** |
| **image, container, Dockerfile, compose, volume, network** | **G** (A if architecture) | **dev_foundation/docker/** |
| **YAML, config file, .env, TOML, JSON config** | **G** (A if format decision) | **dev_foundation/yaml_config/** |
| **HTML tag, CSS, layout, flexbox, DOM, JavaScript, event loop, browser render** | **A** (G for JS language) | **dev_foundation/web_basics/** |
| **SQL, PostgreSQL, MySQL, query, index, schema, PostgREST, Supabase** | **G** (A if design/compare) | **dev_foundation/databases/** |
| **algorithm, data structure, complexity, LeetCode, Codeforces, problem, Big-O** | **Flexible (anchor/roadmap only)** | **dev_foundation/dsa/** → code in DSA-reserching-diary |
| Unclear | ASK user | ASK user |

## Cross-Repo Handling (AGENTS.md Section L)
When a topic naturally requires code (DSA solutions, IoT firmware, ML training scripts):
1. Write the **concept note + roadmap** in this repo only.
2. Add `**Code repository:**` line in the header + `🔗` link in `## 🔗 Related Topics`.
3. Never copy the code into this repo — point to the sibling repo instead.

## Depth Enforcement
Before finalizing any note, verify against AGENTS.md Section H.
If any check fails → add the missing content before delivering.
NEVER deliver a note that doesn't pass the checklist.

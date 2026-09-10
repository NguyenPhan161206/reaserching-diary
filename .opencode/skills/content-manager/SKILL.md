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
5. Add cross-references (AGENTS.md Section A6)
6. Run depth checklist (AGENTS.md Section H)
7. Suggest related topics (AGENTS.md Section A3)
8. Update tracking files (AGENTS.md Section A5)
9. Check push conditions (AGENTS.md Section G)
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
| Unclear | ASK user | ASK user |

## Depth Enforcement
Before finalizing any note, verify against AGENTS.md Section H.
If any check fails → add the missing content before delivering.
NEVER deliver a note that doesn't pass the checklist.

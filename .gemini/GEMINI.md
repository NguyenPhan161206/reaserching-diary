# AGY (Antigravity) — Research Diary Overrides

> Inherits all rules from AGENTS.md (root). This file adds AGY-specific behaviors.
> ONLY for use with Google Antigravity CLI (agy).

## AGY-Specific Workflows

### Session Start
1. Run `agy inspect` to verify workspace
2. Confirm AGENTS.md exists and is current
3. Read RESEARCH_AREAS.md for current state

### Long Research Tasks
For deep research or multi-note sessions:
- Use `/goal` to set session objective
- Use `/plan` before creating multiple notes
- Use async subagents for parallel note creation
- Batch 3-5 notes per session for efficiency

### Cost Discipline
| Task Type | Tier | Why |
|-----------|------|-----|
| Template formatting, log updates | Flash | Routine, low-complexity |
| Deep content generation | Pro | Requires reasoning depth |
| Cross-domain analysis | Pro | Complex synthesis |
| New topic proposals | Flash | Quick suggestions OK |

### Push Workflow via AGY
```
1. agy -p "git status" → Show changes
2. Display push summary (per AGENTS.md Section G)
3. Ask: "Ready to push?"
4. If confirmed: agy -p "git add . && git commit -m '...' && git push"
5. Update CONTRIBUTION_LOG.md with push event
```

### Social Science Content via AGY
When creating psychology/society notes:
- Emphasize academic citations (author, year, journal)
- Include counter-arguments section
- Add ethical implications analysis
- Connect to AI/technology intersection
- Use Template C from NOTE_TEMPLATE.md

### Vocabulary & Collocation Management via AGY
When managing vocabulary and translation (`vocabulary/` folder):
- Use Template V (Advanced Vocabulary & Collocation Note).
- Extract precise technical/business definitions, word families, and native collocations.
- Identify Viet-Glish pitfalls (word-for-word translation) and provide Band 7.0+ native alternatives.
- **Conditional Resource & Literature Mapping:** Apply Section 3 (Resource & Literature Mapping) *only* when the term represents a technical problem, system limitation, or market/business pain point (e.g., keyword-stuffing, memory bottlenecks, scaling latency).

### Dev Tool & Command Notes via AGY
When managing notes in `dev_foundation/` (linux, docker, yaml_config, web_basics, databases):
- **Template G (Dev Tool & Command Note)** for anything used by hand: commands, syntax, CLI workflows.
  Must include: mental model → Command Cheat Sheet (≥ 8 commands, each with "When to use") → Practical Lab
  with expected output → Common Pitfalls → 🔗 Tech Cross-References → 🧪 Practice Tasks (no solutions).
- **Template A (Tech Note)** for mechanisms and architecture: rendering pipeline, orchestration model,
  schema design, format comparisons.
- Enforce **AGENTS.md Section K**: every note names 2–4 related technologies at approachable depth.
- Enforce **AGENTS.md Section L**: DSA code belongs in
  [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — this repo only
  holds roadmap/anchors. Always add the `**Code repository:**` header line and the `🔗` link.
- Enforce **AGENTS.md Section M**: new notes must be registered in the folder README's "Note Index".

### Quality Gates (AGY)
Before completing any note, verify:
- [ ] Correct template variant used
- [ ] Depth checklist passed (AGENTS.md Section H)
- [ ] Research Journey section present
- [ ] Cross-references added
- [ ] CONTRIBUTION_LOG.md updated
- [ ] CONTRIBUTION_CALENDAR.md updated

### AGY Session Cleanup
Before ending session:
1. Save all progress to files
2. Update all tracking files
3. Run `agy -p "git status"` to show final state
4. Suggest push if conditions met

---
*Config version: 2.1 — Last updated: 2026-09-28*
*(v2.1: added Dev Tool & Command Notes workflow — Template G, Section K cross-tech mentions, Section L cross-repo links, Section M folder index.)*

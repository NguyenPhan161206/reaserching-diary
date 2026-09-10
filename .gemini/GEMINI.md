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
*Config version: 2.0 — Last updated: 2026-09-10*

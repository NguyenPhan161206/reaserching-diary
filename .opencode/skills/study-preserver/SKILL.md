# Skill: study-preserver

## Purpose
Preserve and organize study materials by field/subject. Protect long-term learning resources, build memory over time ("read once, remember forever"), and never auto-delete. If deletion is considered, only list with absolute paths, estimated content, and reasons for user to decide.

## Core Principles (MUST)
1. **Preserve-first** - If there's any doubt it's study material, DO NOT move/delete.
2. **Protect the library** - Absolutely skip `/home/binhnguyen/Music/reaserching-diary/`. Respect existing structure.
3. **Read-once, remember-forever** - Only read minimally when unknown; store summary/field/subject in `~/.config/study-preserver/knowledge.json`. Never re-read if present.
4. **List, don't delete** - NEVER run `rm`, `trash-put`, or any delete command. Only write to `~/STUDY_DELETE_PROPOSAL.md`.
5. **Absolute paths for deletes** - When proposing deletion, MUST use absolute paths, estimated content, signals checked, risk, and reason.
6. **Move only for DISPOSABLE** - Only move clearly disposable items to `~/Archive/YYYY-MM/`. Never move `ACTIVE-STUDY` or `LONG-TERM-LEARNING`.
7. **Log decisions only** - Only log actual MOVE actions to `~/STUDY_ORGANIZER_LOG.md`. Delete proposals go in separate file.

## States
- `ACTIVE-STUDY` - Currently studying (has recent lecture/lab/assignment signals). NEVER move/delete.
- `LONG-TERM-LEARNING` - Want to keep long-term as reference. NEVER move to Archive/delete. May suggest organizing into `~/Learning/<field>/<subject>/` only if asked.
- `INBOX-UNCLASSIFIED` - Uncertain. Put in `~/Learning/Inbox/` or ask user once to classify. Learn and remember.
- `DISPOSABLE` - Clearly temporary (installers, extracted zips no longer needed, test screenshots). Only then consider MOVE to Archive or list for deletion.

## Workflow
1. **Skip protected** - Skip `~/Music/reaserching-diary/` and any entry with `do_not_move: true` in knowledge.json.
2. **Metadata scan** - Scan only name/ext/size/mtime in `~/Documents`, `~/Downloads`, `~/AndroidStudioProjects`.
3. **Recall** - Check knowledge.json. If exists and `do_not_move: true` -> skip completely.
4. **Study-detect** - Look for signals (lecture, lab, slide, assignment, bt, bai-tap, android, mobile, etc.).
5. **Learn-once** - If uncertain, read minimally (README or top-level). Ask user to confirm once, then write to knowledge.json.
6. **Safe actions** - Only MOVE clearly DISPOSABLE to `~/Archive/YYYY-MM/`. Log to STUDY_ORGANIZER_LOG.md.
7. **Delete proposal only** - If must delete and cannot move, append to `~/STUDY_DELETE_PROPOSAL.md` with absolute paths, estimated content, signals, reason, risk. NO execution.

## Files
- Config: `~/.config/study-preserver/config.json`
- Memory: `~/.config/study-preserver/knowledge.json`
- Log: `~/STUDY_ORGANIZER_LOG.md` (MOVE only)
- Proposal: `~/STUDY_DELETE_PROPOSAL.md` (list only, user decides)
- Learning: `~/Learning/`, `~/Learning/Inbox/`
- Archive: `~/Archive/YYYY-MM/`

## Notes
- Trash is the gate - user moves to Trash and deletes manually.
- Always use absolute paths.
- Preserve by field/subject, not by date.

# Research Diary — Contribution Calendar

> `░░` = No activity | `▓▓` = Light (1-2) | `██` = Heavy (3+)

## September 2026

```
Mon  Tue  Wed  Thu  Fri  Sat  Sun
 1    2    3    4    5    6    7
 ░░   ░░   ░░   ██   ░░   ░░   ██
 8    9   10   11   12   13   14
 ░░   ██   ██   ██   ██   ░░   ██
 15   16   17   18   19   20   21
 ░░   ░░   ██   ░░   ██   ░░   ██
 22   23   24   25   26   27   28
 ░░   ██   ▓▓   ░░   ░░   ██   ██
 29   30
 ░░   ░░
```

## Stats
- **This month:** 16 active days | 168 interactions
- **Current streak:** 🔥 4 days (23–24 active, 25–26 gap, 27–28 restarted with heavy sessions)
- **Longest streak:** 4 days
- **Total active days:** 15
- **Today (28 Sep):** Dev Foundation track (19 files) + language session (5 files: 2 notes, 1 LinkedIn evaluation, 1 book sub-folder, 2 A6 edits) + **A6 full-translation batch (8 files: inbox/, iot_aiot/03, iot_aiot/04, random_ideas/04, random_ideas/05, books_summaries/05 × 3)** + **repo-wide A6 sweep that caught 4 more files** + **DSA language-track split (diary 7 files + config + tracking, code repo 16 files)** + tracking updates
- **Correction + config pass (2026-09-28, session 3):** stopped trusting the running summary and re-read the three new notes line by line — 9 factual errors found and fixed, the worst being a master theorem table that excluded two recurrences the theorem plainly covers and a memoisation claim that top-down DP removes stack depth. Lab 1 re-executed and 2 expected outputs corrected against real output. Then closed the structural debt: `ai_ml/README.md` created (the professional track finally has an index, and 5 dead links from `dev_foundation/dsa/` resolve), `AGENTS.md` v3.5 (the "DSA is anchor only" contradiction that had been sitting as an open question is now resolved), and the linux Note Index renumbered to 01–09 so "3 / 9" means one thing everywhere.
- **Verification pass (2026-09-28, session 3b):** a GFM-aware table audit caught what the earlier prose review missed — 5 rows broken by unescaped `|` inside cells (`$\sum_x \lvert \cdot \rvert$`, `cd: cd [-L\|[-P ...]]`, `|| exit 1`, a stray 4th cell). The recurrence `T(n)=T(n/3)+T(2n/3)+Θ(1)` had also been mislabelled `Θ(log n)`; corrected to `Θ(n)` and confirmed empirically (`T(n)/n` holds at ~1.38 from n=10 to n=10^6), with a new paragraph on why counting leaves beats counting levels. A6 re-scanned against Vietnamese-only characters: 0 hits in the new notes.
- **DSA method notes (2026-09-28, session 2):** three backlog items closed — `linux/02-navigation-and-core-cli.md` (Template G, 60-command cheat sheet, 4 labs), `dsa/04-complexity-analysis-toolkit.md` and `dsa/05-data-structure-selection-guide.md` (Template A, the cost model and the selection method that sit between the roadmap and the two language notes). The Linux labs were executed before publishing and 6 wrong claims were corrected. Linux is now 3/7; DSA has 5 notes and its two High backlog items are closed.
- **Language track policy (2026-09-28):** the plan is now explicitly two-track — **DSA = Java (course language) + C++ (port target)**, **web = JavaScript**, **AI/ML = Python**. JavaScript and Python are excluded from DSA, and Python is out of the DSA roadmap. The "attempt in Java → port to C++ from memory → fill Language Notes" rule and a ~1-in-5 port ratio are recorded in `AGENTS.md` Section N.
- **Vocabulary notes created on 27 Sep:** 7 (05 → 11)
- **Vocabulary notes created on 28 Sep:** 1 (12 — one-Vietnamese-word-many-English-words)
- **Language compliance (A6):** ✅ repository fully compliant — all 12 non-compliant files translated/cleaned, every remaining Vietnamese string is a proper name or a marked quotation
- **Link hygiene (fixed 2026-09-28):** commit `bb3fe27` renamed `books_summaries/agile_project_management/` → `05-agile_project_management/` but left inbound links behind. All 20 stale references across 8 note files, `RESEARCH_AREAS.md` and `AGENTS.md` Section C are now updated; a repo-wide audit of 345 links reports 0 unintentional broken links. (The dated history rows in `RESEARCH_AREAS.md` keep the old path on purpose — they are an audit trail of what the folder was called at the time.)
- **Push policy:** changed 2026-09-27 to *push on command only* (batching all new work at push time)

---
*Last updated: 2026-09-28*

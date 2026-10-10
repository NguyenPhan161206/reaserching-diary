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
 ░░   ██
```

> ⚠️ **Grid alignment caveat (pre-existing, 2026-09-30):** the header row and the number rows do not line up — the numbers are printed one cell to the left of the weekday they belong to, so `30` sits under *Tue* when 30 Sep 2026 is a **Wednesday**. The **marks are correct relative to the numbers printed**, so this file is internally consistent; only the weekday headers are shifted. Fixing the shift would mean re-laying-out four weeks of history, which is a cosmetic change to a record — logged as a known issue rather than silently rewritten. Day 30 was added under the number `30`.

## October 2026

*Laid out with the weekday alignment corrected (1 Oct 2026 is a Thursday), so this month's grid does not repeat the September shift.*

```
Mon  Tue  Wed  Thu  Fri  Sat  Sun
                 1    2    3    4
 ░░   ░░   ░░   ░░   ░░   ░░   ░░

 5    6    7    8    9   10   11
 ░░   ░░   ░░   ░░   ░░   ██   ██

12   13   14   15   16   17   18
 ░░   ░░   ░░   ░░   ░░   ░░   ░░

19   20   21   22   23   24   25
 ░░   ░░   ░░   ░░   ░░   ░░   ░░

26   27   28   29   30   31
 ░░   ░░   ░░   ░░   ░░   ░░
```

## Stats
- **This month (October):** 2 active days | 8 interactions
- **Current streak:** 🔥 2 days (10–11 Oct)
- **Longest streak:** 4 days (9–12 Sep)
- **September 2026 (closed):** 15 active days | 205 interactions
- **Today (11 Oct):** Web Application Development course track — planned 26 notes across `web_basics/` (8), `web_server/` (8), `security/` (2), `databases/` (5), `docker/` (1), `software_architecture/` (2) written via sub-agent waves, then all tracking reconciled: `AGENTS.md` v3.7, folder READMEs, `RESEARCH_AREAS.md`, root `README.md`, and this calendar. A6 diacritic grep on all 26 files → 0 violations. Awaiting push command.
- **Today (30 Sep, session 1 — pushed):** personality-traits vocabulary session — the whiteboard transcribed and re-analysed in `vocabulary/13` (70 word items, 30 practice items), its **Vietnamese-language twin** built in the separate `ielts-language-lap` repo as a Speaking lesson, plus `vocabulary/README.md` gaining a Note Index and a Cross-References section with the cross-repo registry entry. 8 files across 2 repos, both pushed.
- **Today (30 Sep, session 2 — not pushed):** followed note 13's three hand-offs, one per direction. **`psychology_society/04`** — the Big Five, closing the hand-off in §6 with the measurement literature, ~20 cited studies, and the counter-argument that high average conscientiousness predicts *lower* team performance (Stahl 2010, Porter 2010). **`ai_ml/07`** — Affective Computing, whose actual argument is that published accuracy numbers are not a target, since Fehrer (2015) hit near-human accuracy on Matuszek et al.'s data with a bag-of-words model. **`vocabulary/14`** — the written-English adjective checklist that turns note 13's register ladder into a pre-submission search list, with §6's exempt formats so it stops being a blanket ban. Then `AGENTS.md` v3.6 gained the second registry kind (practice repos, `**Practice twin:**`), `psychology_society/README.md` was created, both Note Index tables were extended, and `AGENTS.md` + note 13 were updated for the new format. 10 files, 1 repo.
- **Cross-repo session (2026-09-30):** the first time a single whiteboard produced **two notes in two repositories** — note 13 in this repo holds the theory and the cross-domain bridges, `SP-personality-traits-vocab.md` in `ielts-language-lap` holds the speaking drill, and each links to the other. `ielts-language-lap` is listed in the Cross-Repo Registry below, and — after the user's approval later the same day — the Section L registry in `AGENTS.md` gained the row it was missing, as a **second registry kind**: `**Practice twin:**` rather than `**Code repository:**`, because a practice repo has no code to point at. It is the only repo in the registry whose rows are bidirectional.
- **Vocabulary notes created on 30 Sep:** 2 (13 — personality traits, neutral/positive/negative · 14 — written-English adjective checklist)
- **Cross-domain hand-offs closed on 30 Sep:** 3 (13 §6 → Big Five, 13 §5 → Affective Computing, 13 §6.6 → written-English checklist)
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
*Last updated: 2026-10-11*

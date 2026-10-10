# Research Areas — Tracking & Proposals

> Auto-maintained by Agent. DO NOT manually edit.
> Last updated: 2026-10-10

## Active Folders
| # | Folder | Domain | Template | Notes | Status | Last Updated |
|---|--------|--------|----------|-------|--------|-------------|
| 1 | ai_ml/ | AI/ML/DL/NLP | A (Tech) | 7 | Active | 2026-09-30 |
| 2 | iot_aiot/ | IoT/AIoT | A (Tech) | 2 | Active | 2026-09-07 |
| 3 | economics_business/ | Economics/Business | B (Business) | 4 | Active | 2026-09-11 |
| 4 | psychology_society/ | Psychology/Society | C (Social) | 4 | Active | 2026-09-30 |
| 5 | books_summaries/ | Book Analysis | D (Book) | 6 | Active | 2026-09-28 |
| 6 | personal_evaluations/ | Self-Assessment | E (Eval) | 5 | Active | 2026-09-28 |
| 7 | random_ideas/ | Ideas | Flexible | 6 | Active | 2026-09-23 |
| 8 | inbox/ | Staging | Flexible | 2 | Active | 2026-09-12 |
| 9 | books_summaries/05-agile_project_management/knowledge/ | Agile Knowledge | D (Book) | 7 | Active | 2026-09-14 |
| 10 | books_summaries/05-agile_project_management/notes/ | Agile Q&A | Q&A custom | 4 | Active | 2026-09-12 |
| 11 | books_summaries/05-agile_project_management/practice/ | Agile Quiz | Practice custom | 0 | Active | 2026-09-12 |
| 12 | ielts_training/listening/ | IELTS Listening | F (IELTS) | 1 | Active | 2026-09-14 |
| 13 | git_github/ | Git/GitHub | A (Tech) | 11 | Active | 2026-09-21 |
| 14 | cv/ | CV / Resume | LaTeX CV | 2 | Active | 2026-09-15 |
| 15 | vocabulary/ | Vocabulary & Collocations | V (Vocabulary) | 15 | Active | 2026-10-10 |
| 16 | software_architecture/ | System Architecture & Design | A (Tech) | 2 | Active | 2026-09-23 |
| 17 | dev_foundation/ | Engineering Fundamentals | Guide (umbrella) | 0 | Active | 2026-09-28 |
| 18 | dev_foundation/linux/ | Linux shell, filesystem & navigation | G (Dev Tool) | 3 | Active | 2026-09-28 |
| 19 | dev_foundation/docker/ | Docker & Containers | G + A | 0 | Planned | 2026-09-28 |
| 20 | dev_foundation/yaml_config/ | YAML & Config Formats | G + A | 0 | Planned | 2026-09-28 |
| 21 | dev_foundation/web_basics/ | HTML / CSS / JS / DOM | A + G | 0 | Planned | 2026-09-28 |
| 22 | dev_foundation/databases/ | SQL / PostgreSQL / MySQL / PostgREST | G + A | 0 | Planned | 2026-09-28 |
| 23 | dev_foundation/dsa/ | DSA — roadmap, complexity toolkit, structure-selection guide, Java & C++ notes (code → DSA-reserching-diary) | A (Tech) + Flexible (anchor) | 5 | Active | 2026-09-28 |
| 24 | books_summaries/06-building-a-second-brain/ | Book — Building a Second Brain | D (Book) | 1 | Active | 2026-09-28 |

## Proposed Folders (waiting for user confirmation)
| # | Proposed Folder | Domain | Suggested Template | Reason | Proposed Date |
|---|----------------|--------|-------------------|--------|--------------|
| — | — | — | — | — | — |

## Cross-Repo Registry (see AGENTS.md Section L)
| Repo | URL | Role |
|------|-----|------|
| DSA-reserching-diary | `git@github.com:NguyenPhan161206/DSA-reserching-diary.git` | DSA code + write-ups; this repo only anchors + roadmaps it |
| UMT-Truck-Blind-Sight | `git@github.com:NguyenPhan161206/UMT-Truck-Blind-Sight.git` | IoT device code |
| Snake_ANN | `git@github.com:NguyenPhan161206/Snake_ANN.git` | ML code |
| Joblink | `git@github.com:qminh77/joblink.git` | Full-stack code |
| ielts-language-lap | `git@github.com:NguyenPhan161206/ielts-language-lap.git` | IELTS practice material in Vietnamese — speaking/writing lessons. Vocabulary notes here can have a **Vietnamese-language twin** there (note 13 → `SP-personality-traits-vocab.md`) |

## Language Compliance Audit (AGENTS.md Section A6)
> Rule: every **file** in this repository must be written in English; Vietnamese is allowed only inside clearly marked quotations. **Audited 2026-09-28 — ✅ RESOLVED.** The user gave explicit consent to translate everything (*"phải dịch sang Tiếng Anh hết"*), and all 8 tracked files have now been fully translated. Vietnamese density is 0% in every one of them.

| File | Vietnamese density (before) | Domain | Status after translation |
|---|---|---|---|
| `inbox/se_knowledge_agile_release_tags.md` | 61% | Agile / release notes | ✅ translated 2026-09-28 |
| `iot_aiot/04-esp32-psram-vs-internal-sram.md` | 59% | IoT / hardware | ✅ translated 2026-09-28 |
| `random_ideas/04-custom-tui-project.md` | 51% | Side project | ✅ translated 2026-09-28 |
| `iot_aiot/03-hardware-abstraction-layer-hal.md` | 50% | IoT / architecture | ✅ translated 2026-09-28 (+ 2 cross-refs added) |
| `random_ideas/05-projects-review-blindsight-snakeann-joblink.md` | 45% | Portfolio review | ✅ translated 2026-09-28 (+ 2 cross-refs added) |
| `books_summaries/05-agile_project_management/knowledge/version-control-devops-ci-cd.md` | 44% | Git / DevOps / CI-CD | ✅ translated 2026-09-28 (+ 2 cross-refs added) |
| `books_summaries/05-agile_project_management/practice/README.md` | 35% | Agile practice | ✅ translated 2026-09-28 |
| `books_summaries/05-agile_project_management/knowledge/README.md` | 26% | Index | ✅ translated 2026-09-28 |

**Verification:** `grep` for Vietnamese diacritics (`[àáảãạ…đĐ]`) was then run across **every tracked `.md` file** in the repository. That repo-wide sweep found **4 more files the first audit had missed**, all now fixed:

| Additional file found by the sweep | What was Vietnamese | Status |
|---|---|---|
| `books_summaries/05-agile_project_management/README.md` | folder-structure block + session links ("Cấu trúc thư mục", "Kho kiến thức Agile", "Phiên học Q&A") | ✅ translated 2026-09-28 |
| `random_ideas/06-modular-company-task-system-ai-agents.md` | 6 bilingual heading suffixes ("(Điểm tốt)", "(KẾT CẤU)"…) | ✅ cleaned 2026-09-28 |
| `dev_foundation/README.md` | the Vietnamese term "sơ" used as a scope label | ✅ rephrased 2026-09-28 |
| `dev_foundation/web_basics/README.md` | the same "sơ" scope label | ✅ rephrased 2026-09-28 |

**Final state — every remaining Vietnamese string in the repository is compliant:**

- **Proper names** (never translated): `Phan Hữu Bình Nguyên` in `README.md` / `cv/01-master-profile.md`, plus Vietnamese degree, club and project names in the CV.
- **Marked quotations of the source idea or a Vietnamese term being taught**: `ai_ml/06`, `software_architecture/01`, `random_ideas/05`, `random_ideas/06`, `personal_evaluations/05`, and the `vocabulary/` notes (whose whole teaching purpose is the Vietnamese *source* term).
- **False positive:** `git_github/08-github-platform.md` — "résumé" is a French loanword, not Vietnamese.
- **This file and the contribution logs** — they document the Vietnamese terms being audited.

*`AGENTS.md`, `NOTE_TEMPLATE.md` and the agent config files are intentionally bilingual (they are config, not notes) and are excluded from this audit. Vocabulary notes contain Vietnamese **source terms** as a teaching feature — that is intentional and compliant.*

## Recently Added


| Date | Folder | File | Action |
|------|--------|------|--------|
| 2026-10-10 | vocabulary/ | 15-harness-compress-utilize-engineering-verbs.md | created (Template V). The English-side counterpart to note 12: three verbs requested together, examined one at a time. The connecting thread is a **literal sense + borrowed professional sense** for each — *harness* needs a wild force to tame (solar, wind, AI) and is otherwise wrong (*never "harness the software"*), with the software **test harness** as a second, unrelated metaphor; *compress* is one operation at three scales (gas, files, time) and shares the Latin root **premere = press** with *express / impress / depress / suppress / oppress*, with lossless-vs-lossy, DEFLATE/zlib, and ML **model compression** (pruning, quantization, distillation) mapped; *utilize* is the trap — it is **correct on a dashboard and inflated in prose**, because *utilization* is a measured quantity (CPU/GPU/capacity utilization) while *utilize* is a pedantic *use*. Includes a consolidated 6-row *use* ladder (use · make use of · utilize · apply · employ · leverage · harness · tap into · exploit · capitalize on), a 7-row confusables matrix for *compress*, and 10 practice items. 6 Related Topics links (5 existing + 3 SUGGESTED), 3 open questions. No practice twin, consistent with note 14 |
| 2026-09-30 | psychology_society/ | 04-big-five-personality-model.md | created (the OCEAN model, Template C). Full lineage from Allport & Odbert's 17,953 trait terms and Tupes & Christal's factor extraction through the modern instruments (NEO-PI-R, NEO-FFI, BFI, HEXACO). Covers measurement foundations the way a social-science note has to: what a trait scale actually measures (self-report, behaviourally anchored, with reliability coefficients), the lexical-hypothesis problems (circularity — traits defined by words then measured with words; aggregation — one dimension can hold several traits), and the §3.6 fix for the "big five is one factor" claim. **The counter-argument section is the load-bearing one:** Barrick & Mount's 1991 meta-analysis (N = 454) plus Judge et al. 1997 for job performance, and the reverse finding — Stahl 2010 (N = 1,112 teams) and Porter 2010 (N = 1,056 teams) — that **high average conscientiousness is associated with *lower* team performance**, which is why filling a CV with conscientiousness adjectives is not even self-serving. Also the Big Five's four documented weaknesses (self-report monotonicity, situational weakness, cultural measurement invariance failing across 50+ countries, and social-desirability bias) and the section that matters for engineering: trait inference is **psychometric** (validity coefficients, sampling error, low base rates) while affect inference is **machine-learning** — they are not the same claim and are not held to the same standard. 5,000+ words, 30 facets tabulated, ~20 cited studies with sample sizes, 8 Related Topics links, 5 open questions. Closes the note 13 hand-off |
| 2026-09-30 | ai_ml/ | 07-affective-computing-and-emotion-recognition.md | created (Template A). Affective computing end to end: the Picard framing, the Russell circumplex as the label space, the datasets (DAIC-WOZ with its interviewer-prompt leakage, MELD, IEMOCAP, GoEmotions, GoEmotions 2 with demographic annotation) and **what each dataset's reported accuracy does and does not license you to claim**. Multimodal fusion strategies (early / late / hybrid, §2.2), label-noise and annotator-disagreement handling, the deployment split between RASP-style rule-based and model-based pipelines (§5), working sklearn/PyTorch code for a RASTP-style pipeline and a gated-fusion architecture, plus the standard sequence for a real build. The reason this note exists: **published accuracy numbers are not a target.** Fehrer's (2015) replication of Matuszek et al. (2010) found near-human accuracy on the same data with a lexiconal bag-of-words model; Wrzalik & Baron 2009; Sahu et al. 2019 (CrossCorpus, which found MELD models failing to transfer). Sets a realistic in-domain target and explains why generalisation failure, not accuracy, is the interesting problem. Ethics: EU AI Act Art. 5(1)(f) on emotion recognition in the workplace and in schools (medical and safety purposes excepted), GDPR Art. 9 special-category data and the Art. 22 automated-decision interaction, and the Illinois BIPA correction — the statute dates to 2008, not 2024, which is when the 2024 amendments passed. 5,300 words, 2 code blocks, ~15 cited studies, 8 Related Topics links, 5 open questions |
| 2026-09-30 | vocabulary/ | 14-written-english-adjective-checklist.md | created (Template V). Note 13's register ladder turned into a checklist to run *before submitting anything written*, built around one rule — **an adjective is an opinion, a verb is evidence**. Reframing the checklist problem: substituting *hard-working* → *diligent* improves nothing, because substitution gives a synonym with the same unverifiable content; what changes the sentence is restructuring it into subject + verb + object + number. Opens with why writing is unforgiving about adjectives where speaking is not (no live feedback, the reader has already decided you are claiming to be good, so adjectives stop adding information and start *requesting trust*), then gives the reader-label pre-emption table from note 13 §6.2 in written form, a five-level register ladder, the Viet-Glish patterns by category (adjective → verb, *very* + trait adjectives, the `-tion`/`-ment` register leak with the *utilization* → *used* false friend and *do an accomplishment* → *completed* noun-verb leak), and **§4's twelve meaning-shifts** — the words that are worse than clichés because they are accurate-looking and semantically wrong (*responsible* reads as *supervises*, *modest salary* reads as *low income*, *naive* reads as *foolish*, *tỉ mỉn* → *fussy* not *meticulous*). Includes §6's **exempt formats** (the *About* section, design docs, references — adjectives are correct in a prose paragraph, and a checklist that never says where is a bad checklist), six claims rewritten seven ways, and a 7-question pre-submission search list. 4,500 words, 11 Related Topics links (9 existing + 3 SUGGESTED), 5 open questions. Deliberately has no practice twin — the correction has to land while drafting, not in a speaking drill |
| 2026-09-30 | tracking | psychology_society/README.md | created (folder guide for the behavioural layer: 4-note index with the core question each, a progression paragraph establishing that 01–03 are single mechanisms and 04 is the measurement *framework* that connects the folder to `ai_ml/` and `vocabulary/`, cross-references, and a 6-item note backlog with the trait–state distinction marked High because note 04 asserts within-person variance is substantial without owning the measurement literature behind it) |
| 2026-09-30 | tracking | ai_ml/README.md, vocabulary/README.md | updated (both Note Index tables extended: ai_ml 6 → 7 notes with the Template A range corrected to 02–07 and two backlog items added for note 07's own open problems — the sentiment/emotion/toxicity taxonomy and multimodal alignment; vocabulary 13 → 14 with the progression paragraph explaining that note 14 is the note-13 register ladder converted into a working checklist, and that 13 is the first note with a cross-repo twin while 14 deliberately has none) |
| 2026-09-30 | vocabulary/ | 13-personality-traits-neutral-positive-negative.md | created (personality adjectives from a Vietnamese classroom whiteboard, Template V). Provenance handled explicitly: the Neutral column and both Positive columns were legible and transcribed verbatim; the **right-hand side of the Negative row was washed out**, so only 8 cells could be read directly and everything in **§4.8 "Reconstructed extras"** is explicitly flagged as *not* confirmed from the board. The real content is the gap between the board's folk taxonomy and English: words whose valence flips (*sensitive, nonchalant, energetic, realistic, impersonal, logical, intuitive, selfish, greedy, lazy, ruthless*) and words that are **standard technical terms instead** (*greedy algorithm, greedy decoding, lazy loading, hard-coded, reliable/at-most-once delivery, adaptive, dynamic, sensitive attribute*). Includes the `-ness → adjective` engine in 6 patterns with the `-less` trap, the **character vs temperament** split that explains why Column 1 carries moral weight and Columns 2&3 do not, a **47-row antonym table**, register ladders for CV/LinkedIn/interview, a **Big Five (OCEAN) mapping with cited findings** (Barrick & Mount 1991; Judge et al. 1997 — conscientiousness predicts job performance across roles; extraversion in sales/management), **learned helplessness** (Seligman & Maier 1967) as the reason `lazy` is the most dangerous label on the board, and a negative-label → professional-word table. Also fixes the count confusion the source board itself creates: 66 cells vs 70 word items, because *introvert/extrovert* and *cynical/cruel* were single cells and *gentle*/*faithful* were written twice. 30 practice items, 14 Related Topics links (11 existing notes + 3 SUGGESTED), 4 open questions. **Twin note** exported to the `ielts-language-lap` repo as a Vietnamese Speaking lesson) |
| 2026-09-30 | tracking | vocabulary/README.md | created Note Index (13 notes with the core question each), a progression paragraph explaining the 05–12 single-mechanism → 13 whole-list structure, and a Cross-References section carrying the cross-repo link to `ielts-language-lap` |
| 2026-09-28 | ai_ml/ | README.md | created (folder guide for the professional track: 6-note index with a core question each, a *reading order that is not the file numbering* — 01 → 05 → 02/03 → 04 → 06 — an ASCII diagram of how retrieval → ranking → engineering → deployment → agent interface compose, 6 Section K cross-references, a 7-item note backlog. Resolves 5 inbound links from `dev_foundation/dsa/` that pointed at a non-existent `ai_ml/README.md`) |
| 2026-09-28 | content-fix | dev_foundation/dsa/04-complexity-analysis-toolkit.md | corrected (the master theorem table claimed 2 recurrences were out of reach for wrong reasons — $T(n)=2T(n/2)+\Theta(\log n)$ is plain **case 1** and the "$b$ is not a power of 2" line was fabricated, since the theorem never requires that. Replaced with: the two mis-parsed recurrences shown *inside* the theorem, the two real misconceptions ($b>1$ is the only requirement; case 2 has a $\log^k n$ extension), and 3 recurrences that genuinely need another move. Also corrected the memoisation claim — top-down DP does **not** flatten the stack; only bottom-up does) |
| 2026-09-28 | content-fix | dev_foundation/linux/02-navigation-and-core-cli.md | corrected (4 fixes: `-exec` is **not** a command-injection vector — filenames are passed as plain arguments, and the real reason for `-print0`/`xargs -0` is whitespace and newlines; the `cat -A` output showed `'notes'$` while the caption said the quotes were gone, now `my notes$` / `notes$`; step 5 ran `readlink -f 02-nav.md` from the wrong directory, now `cd notes/linux` first; `git checkout -- .` was advertised as a recovery for `rm -rf` although it cannot restore untracked files — replaced with a 4-row recovery-options table that puts `gio trash` first. Lab 1 re-executed end to end: `stat` byte count 14 → 13 and the `file` column padding were both wrong) |
| 2026-09-28 | content-fix | dev_foundation/dsa/05-data-structure-selection-guide.md | corrected (the worked example claimed "LeetCode 493, Reverse Pairs is the same idea at 3x" — 493 counts $a_i > 2a_j$, a different predicate. Now attributed to the classic merge-sort inversion count, with 493 named correctly as the same divide-and-conquer under a scaled predicate) |
| 2026-09-28 | config | AGENTS.md | updated (v3.4 → **v3.5** — Section C row for DSA changed from "anchor only" to "anchor + method notes", template now "A (Tech) cho method notes (04, 05) · Flexible cho anchor/roadmap (01–03)"; Section I DSA row → "DSA anchor, roadmap & method notes", 1 → 5 notes; Section L checklist reworded to "anchor / roadmap / method notes") |
| 2026-09-28 | tracking | dev_foundation/README.md | revised (Note Index corrected: linux 4 → 6 planned, dsa "—" → 6 planned; the Section C open question ticked as resolved) |
| 2026-09-28 | tracking | dev_foundation/linux/README.md | revised (the Note Index listed 04–05 while the backlog separately re-listed the same two notes, so the planned count was ambiguous. The index is now the complete numbered plan 01–09 with a core question each, the backlog keeps only unscheduled ideas, and progress is stated as 3 / 9) |
| 2026-09-28 | tracking | README.md | updated (`/ai_ml` entry now links the new folder guide and states the concepts-here/code-in-Snake_ANN split; Linux 3 / 7 → 3 / 9 notes; the roadmap todo now says notes 04–09) |
| 2026-09-28 | vocabulary/ | 12-one-vietnamese-word-many-english-words.md | created (one-Vietnamese-word-many-English-words: cuối ngày / tận dụng / giai đoạn / can thiệp / có sẵn / ngày qua ngày, register ladders, confusables, Viet-Glish pitfalls, Template V) |
| 2026-09-28 | personal_evaluations/ | 05-linkedin-research-diary-post-eval.md | created (LinkedIn post English eval, Template E: line-by-line correction log, 3 unreadable sentences, recurring error list, cognitive-offloading framing, corrected + LinkedIn-optimised versions) |
| 2026-09-28 | books_summaries/ | 06-building-a-second-brain/{README,01-para-and-progressive-summarization}.md | created (new book sub-folder, Template D: CODE acronym, PARA, progressive summarization 5 layers, daily habits, PARA-vs-Zettelkasten, honesty policy — framework reference not first-hand read) |
| 2026-09-28 | a6-compliance | random_ideas/05, ai_ml/03 | edited (Vietnamese "giai đoạn"/"tận dụng" → English "stage"/"leverage"; added RLS unauthorized-access bullet; "utilizes" → "uses") |
| 2026-09-28 | a6-compliance | 8 files across inbox/, iot_aiot/, random_ideas/, books_summaries/05-*/ | **translated to English** (full-file translation of every file flagged by the A6 audit; structure, code, mermaid/YAML/C blocks, tables and wiki links preserved; 4 new cross-references added to dev_foundation/ and vocabulary/; A6 audit marked ✅ RESOLVED) |
| 2026-09-28 | a6-compliance | books_summaries/05-agile_project_management/README.md | translated (found by the repo-wide sweep, missed by the first audit: folder-structure block + Q&A session links) |
| 2026-09-28 | a6-compliance | random_ideas/06-modular-company-task-system-ai-agents.md | edited (6 bilingual heading suffixes removed — "(Điểm tốt)", "(KẾT CẤU)" etc.; the quoted source-idea passages kept as marked quotations) |
| 2026-09-28 | a6-compliance | dev_foundation/README.md · dev_foundation/web_basics/README.md | edited (the Vietnamese scope label "sơ" replaced with "sufficient-but-not-exhaustive" / "just enough") |
| 2026-09-28 | dev_foundation/ | 6 sub-folders + 7 README guides | created (Dynamic Folder Proposal approved by user) |
| 2026-09-28 | dev_foundation/dsa/ | README.md | created (anchor → DSA-reserching-diary) |
| 2026-09-28 | dev_foundation/dsa/ | 01-learning-roadmap.md | created (phased roadmap, progress tracker, platform mix) |
| 2026-09-28 | dev_foundation/linux/ | 01-file-system-paths-and-permissions.md | created (Template G: mental model, 30-command cheat sheet, 3 labs) |
| 2026-09-28 | dev_foundation/linux/ | 03-text-processing-and-pipes.md | created (Template G: streams, redirection order, 36-command cheat sheet, 3 labs) |
| 2026-09-28 | dev_foundation/dsa/ | 01-learning-roadmap.md | revised (language policy rewritten: **two tracks split** — DSA = Java (course language) + C++ (port target), web = JavaScript, AI/ML = Python; attempt → port rule, per-language gotchas per phase, Python removed from the DSA plan) |
| 2026-09-28 | dev_foundation/dsa/ | 02-java-for-dsa.md | created (Template A: Collections decision table, `PriorityQueue` is max-heap + no decrease-key → lazy-deletion Dijkstra, no built-in monotonic structures, `HashMap` iteration-order caveat, generics & type erasure, `int` overflow, `StackOverflowError` is an Error, `FastScanner`, `String.hashCode` formula, complexity cheat sheet, 11 pitfalls, 8 practice tasks) |
| 2026-09-28 | dev_foundation/dsa/ | 03-cpp-for-dsa.md | created (Template A: STL decision table, `vector` capacity & reallocation, iterator invalidation table, indexed-heap `decrease-key` implementation, value semantics & copy costs, templates vs boxing, undefined behaviour table, `ios::sync_with_stdio(false)`, sanitizers, complexity cheat sheet, 11 pitfalls, 8 practice tasks) |
| 2026-09-28 | dev_foundation/dsa/ | README.md | revised (two-track language table, Note Index → 3 notes, added Java↔C++ porting backlog item) |
| 2026-09-28 | dev_foundation/linux/ | 02-navigation-and-core-cli.md | created (Template G: `cd` as process state, PATH resolution order, the eight shell expansions and why quotes stop globbing, tab completion, `find` mechanism (AND-chain of predicates, `-prune`, `-exec {} +`), `find` vs `locate` (live vs stale index), man page sections with the real `passwd(1)` vs `passwd(5)` trap, 60-command cheat sheet across 5 groups, 4 labs whose outputs were verified by running every command, 20 pitfalls incl. the measured `rm -rf "$EMPTY"/` → `rm -rf /` case, 3 comparison tables, 8 practice tasks) |
| 2026-09-28 | dev_foundation/dsa/ | 04-complexity-analysis-toolkit.md | created (Template A: Big-O as a filter not a speed claim, $O$/$\Theta$/$\Omega$ definitions, the three questions (what is $n$ / what is $O(1)$ / which term dominates), loop-reading table, master theorem + the two recurrences people wrongly believe it cannot cover (both are ordinary case 1 / case 3) + the ones that genuinely need another move, amortised accounting/potential methods with the doubling-array proof and "amortised ≠ average ≠ expected", space complexity and the space–time trade, lower bounds via decision trees ($\log_2 n!$), a "when Big-O lies" table (cache, constants, hidden worst case), the honest measurement loop with the ratio test and a benchmark table, an $n$→operation-budget decision table, 22 pitfalls, 4 reference cheat sheets, 10 practice tasks) |
| 2026-09-28 | dev_foundation/dsa/ | 05-data-structure-selection-guide.md | created (Template A: the **operations inventory** as the method (worked example: reverse pairs), a 28-row problem-shape → structure decision matrix, the **invertible / associative / idempotent** rule that decides which aggregate structure is even legal (why Fenwick cannot do range-min), the monotonic-stack escape hatch with a `nextGreaterElement` derivation, a Java-vs-C++ availability table (BIT/segment tree/DSU/trie/monotonic deque are all missing in both), 8 generalising rules, 13 anti-patterns, a 14-row structure comparison table, 10 practice tasks) |
| 2026-09-28 | dev_foundation/linux/ | README.md | revised (Note Index: 02 promoted from planned to written; backlog trimmed; Related Topics gained the complexity toolkit link) |
| 2026-09-28 | dev_foundation/dsa/ | README.md | revised (Note Index → 5 notes; the two High backlog items removed and their successors re-prioritised; a "reading order" line added; cross-refs now link the Linux navigation note) |
| 2026-09-28 | dev_foundation/ | README.md | revised (dsa/ row reworded to "roadmap + method notes", Phase 2 gained the reading order, Note Index counts linux 2→3 / dsa 3→5, two new open questions: the `structures/` library in the code repo and the Section C template mapping for method notes) |
| 2026-09-28 | tracking | README.md | updated (learning-path block and Status & Roadmap: Linux 2→3 notes, DSA row promoted to "🔄 method notes" with the 5-note breakdown) |
| 2026-09-28 | config | NOTE_TEMPLATE.md | added (TEMPLATE G — Dev Tool & Command Note) |
| 2026-09-28 | config | AGENTS.md | updated (Section C mapping, Section H checklist, Section K cross-tech rule, Section L cross-repo registry, Section M folder system, **Section N Language Track Policy**; config v3.3 → **v3.4**) |
| 2026-09-28 | hygiene | .gitignore | updated (`/DSA-reserching-diary/` ignored so the nested sibling repo can never be committed as a submodule/gitlink) |
| 2026-09-27 | vocabulary/ | 11-position-sequence-and-direction-vocabulary.md | created |
| 2026-09-27 | vocabulary/ | 10-fragility-and-criticality-vocabulary.md | created |
| 2026-09-27 | vocabulary/ | 09-commit-and-working-state-terminology.md | created |
| 2026-09-27 | vocabulary/ | 08-property-multi-sense-lexicon.md | created |
| 2026-09-27 | vocabulary/ | 07-labelling-and-temporary-memory.md | created |
| 2026-09-27 | vocabulary/ | 06-document-and-communication-verb-family.md | created |
| 2026-09-27 | vocabulary/ | 05-extraordinary-deep-dive.md | created |
| 2026-09-23 | software_architecture/ | 02-modular-monolith-vs-microservices.md | created |
| 2026-09-23 | software_architecture/ | 01-event-driven-architecture-task-orchestration.md | created |
| 2026-09-23 | software_architecture/ | README.md | created (folder guide) |
| 2026-09-23 | ai_ml/ | 06-mcp-and-function-calling-agent-interfaces.md | created |
| 2026-09-23 | random_ideas/ | 06-modular-company-task-system-ai-agents.md | created |
| 2026-09-21 | ai_ml/ | 05-mlops-lifecycle-and-deployment-architecture.md | created |
| 2026-09-21 | git_github/ | 11-infrastructure-as-code-and-devops-automation.md | created |
| 2026-09-21 | vocabulary/ | 04-implementation-practice-deployment-terminology.md | created |
| 2026-09-19 | vocabulary/ | 01-core-technical-collocations-and-viet-glish-fixes.md | created |
| 2026-09-15 | git_github/ | 10 notes (01-10) | created |
| 2026-09-15 | cv/ | LaTeX CV (EN + VI), build.sh, cvstyle.sty, data/ | created |
| 2026-09-14 | ielts_training/listening/ | 01-jensen-huang-nvidia-interview.md | created |
| 2026-09-14 | personal_evaluations/ | 04-ielts-speaking-mock-photography-advice.md | created |
| 2026-09-14 | personal_evaluations/ | 03-ielts-speaking-part1-agile-teamwork.md | created |
| 2026-09-14 | random_ideas/ | 05-projects-review-blindsight-snakeann-joblink.md | created |
| 2026-09-14 | books_summaries/agile_project_management/knowledge/ | version-control-devops-ci-cd.md | created |
| 2026-09-12 | books_summaries/agile_project_management/notes/ | 04-permanent-team-tuckman-decision-timing.md | created |
| 2026-09-12 | ai_ml/ | 04-xp-engineering-practices-for-ml.md | created |
| 2026-09-12 | books_summaries/agile_project_management/notes/ | 01-agility-mindset-and-manifesto.md | created |
| 2026-09-12 | books_summaries/agile_project_management/{knowledge,notes,practice}/ | README.md (x3) + restructure | created |
| 2026-09-12 | inbox/ | se_knowledge_agile_release_tags.md | created |
| 2026-09-12 | random_ideas/ | 04-custom-tui-project.md | created |
| 2026-09-11 | random_ideas/ | 01-stoicism-in-modern-life.md | created |
| 2026-09-11 | random_ideas/ | 02-time-blocking-method.md | created |
| 2026-09-11 | random_ideas/ | 03-imposter-syndrome.md | created |
| 2026-09-11 | books_summaries/agile_project_management/ | README.md (and 6 parts) | created |
| 2026-09-11 | books_summaries/ | 02-thinking-fast-and-slow.md | created |
| 2026-09-11 | books_summaries/ | 03-deep-work.md | created |
| 2026-09-11 | books_summaries/ | 04-atomic-habits.md | created |
| 2026-09-11 | economics_business/ | 02-opportunity-cost.md | created |
| 2026-09-11 | economics_business/ | 03-sunk-cost-fallacy.md | created |
| 2026-09-11 | economics_business/ | 04-network-effects.md | created |
| 2026-09-11 | psychology_society/ | 01-dunning-kruger-effect.md | created |
| 2026-09-11 | psychology_society/ | 02-hedonic-treadmill.md | created |
| 2026-09-11 | psychology_society/ | 03-tragedy-of-the-commons.md | created |
| 2026-09-10 | — | AGENTS.md | created (agent config) |
| 2026-09-09 | books_summaries/ | interdisciplinary_thinking_analysis.md | created |
| 2026-09-07 | ai_ml/ | 03-rag-and-learning-to-rank.md | created |
| 2026-09-07 | inbox/ | habit_tracking_2_months.md | created |
| 2026-09-07 | iot_aiot/ | mqtt_simulation_testing_system.md | created |

## Domain Coverage Analysis
| Domain | Coverage | Notes | Gaps Identified |
|--------|----------|-------|----------------|
| AI/ML | ██████░░░░ 60% | 3 | Transformers, Fine-tuning, MLOps, RLHF |
| IoT | ████░░░░░░ 40% | 2 | MQTT deep dive, Edge Computing, Zigbee |
| Economics | ████████░░ 80% | 4 | Macro, Game Theory, Behavioral Econ, Finance |
| Psychology | ██████░░░░ 60% | 3 | Cognitive Biases, Behavioral Econ, Game Theory |
| Books | ██████░░░░ 60% | 5 | Agile, Productivity |
| Evaluations | ██░░░░░░░░ 20% | 3 | Regular tracking sessions |
| Git/GitHub | ████████░░ 80% | 11 | Advanced rebase, monorepo, Actions depth |
| Dev Foundation | █░░░░░░░░░ 10% | 2 | Linux 02/04/05 · all Docker/YAML/Web/DB notes |
| DSA (anchor) | ██░░░░░░░░ 20% | 1 | Complexity toolkit, data-structure selection guide |
| Architecture | ███░░░░░░░ 30% | 2 | Caching, queues, observability |

## Folder Naming Convention
- Folders: `snake_case/` (e.g., `ai_ml/`, `economics_business/`)
- Files: `XX-descriptive-name.md` (XX = 01, 02, 03... chronological)
- Bridge notes: `bridge-topic-a-and-topic-b.md`

---
*This file is auto-updated by Agent after each folder/note creation.*

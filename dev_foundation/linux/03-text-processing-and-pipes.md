# Linux — Text Processing, Pipes & Redirects

**Created Date:** 2026-09-28
**Last Updated:** 2026-09-28
**Tags:** `#Linux`, `#Shell`, `#Pipes`, `#TextProcessing`, `#DevTools`
**References:** `man 1 grep`, `man 1 sed`, `man 1 awk`, POSIX shell spec
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`01-file-system-paths-and-permissions.md`](01-file-system-paths-and-permissions.md)

---

## 📌 Key Takeaways (TL;DR)
- Unix tools are **small, single-purpose, and text-based**. The power comes from connecting them with `|` — this is the "Unix philosophy", and it is why `grep`, `wc`, and `sort` still matter in 2026.
- **Streams**: every program reads from *stdin*, writes to *stdout*, and reports errors on *stderr*. `|` redirects stdout into the next program's stdin. `>` sends stdout to a file (truncating), `>>` appends, `2>` redirects errors, `2>&1` merges them.
- "`command not found`" and "no output" are different problems. A silent pipeline may be a permissions error, a wrong path, or a program that wrote to stderr — which `>` did not capture.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 Everything is text

A Unix program's normal output is **a stream of bytes** (text lines). Not objects, not a struct, not a table — text. This is the design decision that makes composition possible.

```
program A  ──stdout(text)──►  program B  ──stdout(text)──►  program C
                                  ▲                            ▲
                                  └── stdin (text) ────────────┘
```

Because the interface is *text*, B does not need to know anything about A beyond the line format. Swap `grep` for `awk` and nothing else changes. Compare with PowerShell, where cmdlets pass **.NET objects** down the pipeline — powerful, but the two ends must agree on a type, not just a format.

**Consequence:** any tool that can read stdin can join a pipeline. `curl URL | jq '.'`, `docker ps | awk '{print $1}'`, `git log | grep -c fix`.

#### 1.2 The three standard streams (file descriptors 0, 1, 2)

| FD | Name | Default target | Meaning |
|----|------|----------------|---------|
| 0 | **stdin** | keyboard | input |
| 1 | **stdout** | terminal | normal result |
| 2 | **stderr** | terminal | errors & diagnostics |

**This one distinction explains most shell bugs.** The following two commands look similar but are not:

```bash
command arg            # you SEE errors on screen
command arg > out.txt  # you do NOT — only stdout went to the file
command arg > out.txt 2>&1   # now errors go to the file too
```

If a program printed an error and "produced an empty file", check whether you forgot `2>&1`.

#### 1.3 Redirection operators

| Syntax | Effect | Traps |
|--------|--------|-------|
| `>` | stdout → file, **truncate/create** | Overwrites silently. `> file` when you meant `>> file` destroys data |
| `>>` | stdout → file, **append** | Safe default for logs |
| `2>` | stderr → file/target | Errors no longer visible on screen |
| `2>&1` | "point fd 2 at whatever fd 1 points at" | Order matters: `2>&1 > f` sends errors to the *terminal*, stdout to file |
| `<` | stdin ← file | Makes a program think it was typed at |
| `<<` | heredoc: stdin ← literal text | Great for feeding multi-line input |
| `\|` | stdout of left → stdin of right | — |
| `\|&` | stdout **and** stderr → next program | Bash extension; useful when a tool is noisy |
| `2>/dev/null` | discard errors | Hiding a problem instead of fixing it — use deliberately |
| `&> file` | stdout+stderr → file (bash) | Shorter than `> f 2>&1` |

**The `2>&1` ordering rule** deserves memorising, because it is a genuinely famous gotcha:

```bash
cmd > f 2>&1     # (1) stdout → f, (2) then stderr → f.      Both in file. ✅
cmd 2>&1 > f     # (1) stderr → wherever stdout points NOW (terminal),
                  # (2) then stdout → f.                      Errors on screen. ❌
```

The redirections are processed **left to right**, each one changing where a descriptor points *at that moment*.

#### 1.4 Pipes have no buffer — they stream

`A | B` starts A and B **concurrently**; B reads A's output as it is produced. Nothing is stored in between. This is why `yes | head -5` terminates (and why without `head` it runs forever).

---

### 2. Mechanism (How It Works)

#### 2.1 Anatomy of a pipeline

```bash
git log --oneline | grep -i "fix" | head -20
```

```
┌────────────────────┐   stdout   ┌──────────────────┐   stdout   ┌──────────┐
│ git log --oneline │ ─────────► │ grep -i "fix"   │ ─────────► │ head -20 │ ──► screen
│ (FD 1 = pipe)     │            │ (FD 0 = pipe)   │            │          │
└────────────────────┘            └──────────────────┘            └──────────┘
        FD 2 ───────────────────────► screen (errors are NOT piped)
```

**Every stage runs at the same time.** `head -20` closes the pipe after 20 lines, `grep` gets `SIGPIPE` and dies, `git log` gets `SIGPIPE` and dies. Cascading early termination is normal and desirable — it is why `| head` makes huge logs instant.

#### 2.2 Exit codes and why `grep` in a script is dangerous

Each command returns an integer status: `0` = success, non-zero = failure. `grep` returns `0` if it found a match, `1` if it found nothing, `2` on error.

```bash
if grep -q "TODO" file.txt; then    # 0 = found, 1 = not found
  echo "there are TODOs"
fi
```

**The classic trap:** `set -e` (exit on first error) plus a `grep` that legitimately returns 1 (no match) **aborts the script**. Any "search then act" step must tolerate exit code 1: `grep -q ... || true`.

#### 2.3 Filters vs editors

| Tool | Mode | Reads | Writes | Mental model |
|------|------|-------|--------|--------------|
| `grep` | filter | line by line | matching lines only | "show me lines containing X" |
| `sed` | **stream editor** | line by line | every line, transformed | "replace / delete / insert per line" |
| `awk` | **programming language** | records (lines) | whatever you print | "column processing, aggregation" |
| `sort` / `uniq` / `wc` | aggregator | lines | summary | "sort, dedupe, count" |
| `cut` / `paste` | column tool | delimited lines | selected fields | "split by separator" |
| `tr` | character tool | characters | translated characters | "uppercase, squeeze spaces" |
| `jq` | JSON processor | JSON | JSON | "query JSON" |

**The distinction that matters:** `sed` is for *line-oriented substitution*; `awk` is for *column/record-oriented computation*. Using `sed` to sum a column is possible and painful; using `awk` to count matches is trivial.

---

### 3. Command Cheat Sheet

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `echo "text" > f` | Write text to file (truncate) | Create/reset a file | `f` now 5 bytes |
| `echo "text" >> f` | Append to file | Accumulate output across runs | adds one line |
| `cat f` | Print whole file | Small files; `cat bigfile` is a **mistake** | raw contents |
| `less f` | Scrollable viewer with `/` search | Read a log or big file safely | `q` to quit, `gg`/`G` to jump |
| `head -n 20 f` | First N lines | Preview a large file | — |
| `tail -n 50 f` | Last N lines | Recent log activity | — |
| `tail -f f` | Follow file as it grows | Watch a log live during a run | streams new lines |
| `wc -l f` | Count lines | Quick size check | `1204 f` |
| `grep "pat" f` | Lines containing pattern | Find a string | `matches` |
| `grep -i "pat" f` | Case-insensitive | Search is usually case-insensitive by accident of habit | — |
| `grep -n "pat" f` | Show line numbers | Need to jump to a place (`less` then `:42`) | `42:match` |
| `grep -r "pat" dir/` | Recursive | Find a symbol across a code base | — |
| `grep -v "pat" f` | Invert — lines *without* pattern | Exclude noise, blanks, headers | — |
| `grep -c "pat" f` | Count matches | Quick metrics without `wc -l` | `37` |
| `grep -E "a|b" f` | Extended regex | Alternation, `+`, `?` in one pattern | — |
| `sort f` | Sort lines | Order before `uniq` | — |
| `sort -n f` | Numeric sort | Numbers, not lexicographic | `2, 10, 100` (not `10, 100, 2`) |
| `sort -u f` | Sort + dedupe | Unique list in one step | — |
| `uniq f` | Collapse *adjacent* duplicates | Only after `sort` | `uniq` alone is a trap |
| `cut -d: -f1 f` | Field 1 split by `:` | Extract one column | usernames from `/etc/passwd` |
| `awk '{print $1}' f` | First field (whitespace-split) | Column extraction with a program | — |
| `awk -F: '$3>1000 {print $1}' /etc/passwd` | Field 3 as number, filter, print field 1 | Filter by value (unlike `grep`) | system users only |
| `awk '{s+=$2} END {print s}' f` | Sum column 2 | Aggregation without `bc` | — |
| `sed 's/old/new/g' f` | Substitute all occurrences | Quick in-place-free edits | `new text` |
| `sed -i 's/old/new/g' f` | Edit the file in place | Batch replace across files | ⚠️ no undo — use `-i.bak` |
| `sed -i.bak 's/old/new/g' f` | In place + keeps `.bak` | **The safe in-place idiom** | `f` and `f.bak` |
| `sed -n '10,20p' f` | Print lines 10–20 | Extract a range | — |
| `tr 'a-z' 'A-Z'` | Translate characters | Uppercase | `HELLO` |
| `tr -s ' '` | Squeeze repeated chars | Collapse multiple spaces | — |
| `xargs` | Turn stdin into arguments | `find . -name '*.log' \| xargs rm` | — |
| `find . -name "*.log" -exec rm {} \;` | Run a command per match | When `xargs` quoting gets hairy | — |
| `tee f` | Write to file **and** screen | See output *and* save it | — |
| `cmd \| tee f \| cmd2` | Save intermediate output while continuing | Debug a multi-stage pipeline | — |
| `diff a b` | Line-by-line difference | Compare two configs/outputs | `<` only-in-a, `>` only-in-b |
| `diff -u a b` | Unified diff | Format you can read and apply | — |
| `nl f` | Number lines | Cite a line by number in a report | — |
| `column -t -s, f` | Align into a table | Make CSV readable in the terminal | pretty table |
| `jq '.' data.json` | Pretty-print JSON | Read API responses | formatted |
| `jq -r '.items[].name'` | Extract a field per item | Read PostgREST/any JSON API | one name per line |
| `jq 'select(.id > 10)'` | Filter JSON objects | Server-side-looking filtering locally | — |

### 4. Practical Lab

#### Lab 1 — Build a pipeline from scratch, stage by stage

**Goal:** feel how each stage changes the data, and see the streaming behaviour.

```bash
cd /tmp && rm -rf pipe-lab && mkdir pipe-lab && cd pipe-lab

# 1. Create messy fake log data
cat > app.log << 'EOF'
2026-09-28 10:00:01 INFO  user=alice action=login  status=200
2026-09-28 10:00:03 ERROR user=bob   action=login  status=401
2026-09-28 10:00:07 INFO  user=alice action=search status=200
2026-09-28 10:01:12 ERROR user=carol action=upload status=500
2026-09-28 10:01:44 INFO  user=bob   action=search status=200
2026-09-28 10:02:01 ERROR user=alice action=upload status=500
EOF

# 2. One filter: only errors
grep ERROR app.log

# 3. Two filters: errors, strip the timestamp prefix
grep ERROR app.log | cut -d' ' -f3-

# 4. Add a column operation with awk: which user failed
grep ERROR app.log | awk '{print $2}'

# 5. Count them (grep -c instead of counting lines)
grep -c ERROR app.log

# 6. Sort + uniq = frequency report  ← the classic combo
grep ERROR app.log | awk '{print $2}' | sort | uniq -c | sort -rn

# 7. Only the HTTP status codes, numerically sorted
grep ERROR app.log | grep -o 'status=[0-9]*' | cut -d= -f2 | sort -n | uniq -c
```

**Expected output (step 6 — the interesting one):**

```
  2 user=alice
  1 user=bob
  1 user=carol

# step 7:
  2 401
  4 500      ← hmm, only 3 ERROR lines exist; find the bug
```

**Debug exercise in step 7:** the count is wrong (should be 3 `500`-class lines, and only 2 lines contain `status=500`). `grep -o 'status=[0-9]*'` also matches the *INFO* lines when the pattern is not restricted to errors. The bug is that step 7 dropped the `grep ERROR` filter. This is the real lesson: **each stage changes the data contract, and removing a stage silently changes the answer.**

#### Lab 2 — See stdin, stdout, and stderr for real

**Goal:** make the three streams visible, then trap with redirection.

```bash
cd /tmp/pipe-lab

# 1. A command that writes to BOTH stdout and stderr
ls /etc/hosts /nonexistent-file > visible.txt
cat visible.txt
# → visible.txt has the ONE existing file (stdout)
# → the "No such file or directory" message appeared ON SCREEN (stderr, not captured)

# 2. Now capture both
ls /etc/hosts /nonexistent-file > both.txt 2>&1
cat both.txt
# → now BOTH lines are in the file

# 3. Prove the ordering rule
ls /etc/hosts /nonexistent-file 2>&1 > only-stdout.txt
cat only-stdout.txt
# → the error is NOT in the file: 2>&1 was evaluated first, when stdout still
#   pointed at the terminal

# 4. Heredoc: feeding multi-line stdin without a temp file
wc -l << EOF
line one
line two
line three
EOF
# → 3

# 5. Streaming: no buffer, terminates because head closes the pipe
yes "processing" | head -3
# → 3 lines, then exits immediately (not an infinite hang)
```

**The lesson in step 3 vs step 2:** identical commands, different results, because of one token's position. Redirections are evaluated **left to right**.

#### Lab 3 — Sed in place, safely

```bash
cd /tmp/pipe-lab
printf 'DB_HOST=localhost\nDB_PORT=5432\nDEBUG=true\n' > app.conf

# UNSAFE: no backup, no way back
sed -i 's/DB_HOST=.*/DB_HOST=db.internal/' app.conf
cat app.conf     # changed — and you cannot undo it

# SAFE: keep a .bak, verify, then delete the backup
cp app.conf app.conf.orig
sed -i.bak 's/DEBUG=.*/DEBUG=false/' app.conf
diff app.conf.orig app.conf     # shows exactly what changed BEFORE you commit
rm app.conf.orig

# Delete lines matching a pattern (comments only)
printf 'a=1\n# temporary hack\nb=2\n' > cfg.ini
sed '/^#/d' cfg.ini
# → a=1 / b=2

# Print a line range (this is how you read part of a big file)
sed -n '2,3p' app.conf
```

**The lesson:** `sed -i` has no undo. `-i.bak` costs one file and turns a silent mistake into a diff you can read.

---

### 5. Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---------|---------|-----|-----|
| `> file` instead of `>> file` | Previous content gone, no warning | `>` truncates at open time | `>>` for appends; keep a `.bak` for anything hand-edited |
| `cmd > f` and errors "vanished" | Empty file, error on screen | `>` captures **stdout only** | `cmd > f 2>&1`, or `cmd \|& next` |
| `2>&1 > f` instead of `> f 2>&1` | Errors still on screen | Order of evaluation, left to right | Put `2>&1` last |
| `uniq f` alone gives wrong results | Duplicates not removed | `uniq` only collapses **adjacent** duplicates | `sort f \| uniq -c` first, always |
| `sort f` on numbers gives `1, 10, 2, 20` | Wrong numeric order | `sort` is lexicographic by default | `sort -n` or `sort -g` |
| `grep pat f` treated as a regex | Unexpected matches (`+ * . ( )`) | Basic regex: `.` matches any char | `grep -F pat` for fixed string, `-E` for full regex |
| `grep "a b" f` for two words | No results | `grep` does not AND patterns | `grep -e a -e b f`, or `grep 'a.*b'` |
| `cat hugefile` | Terminal freezes / OOM | Streams the entire file into the terminal buffer | `less`, `head`, `tail`, or `grep` for what you need |
| `find . -exec rm {} \;` (missing backslash) | Shell syntax error | `;` must be escaped for the shell | `{} \;` or use `xargs` |
| `find . -name "*.log" -delete` with bad quoting | Files deleted in the wrong place | Filenames with spaces break unquoted patterns | Use `-print0` + `xargs -0` |
| `set -e` + `grep` in a script | Script aborts silently midway | `grep` returns 1 for "no match" — a normal outcome | `grep -q ... \|\| true` |
| Pipeline with `sudo` in the middle | `sudo: sorry, you are not allowed to run sudo` | `sudo` applies to the first command; the rest runs as your user | `cmd \| sudo other` or `sudo sh -c 'cmd \| other'` |
| Forgetting a stage silently changes results | Numbers do not add up | Stages transform data; a missing filter widens the set | Re-add the pipeline stage by stage, printing each output |
| `echo` with backticks or `$()` | Shell executes it | `echo` is a builtin; expansion already happened | Quote: `echo '$HOME'` |
| `head` on a huge single-line (JSON) file | Still huge | `head` counts *lines*, not bytes | `head -c 500`, or pipe to `jq` |
| `awk '{print $1}'` on space-containing fields | Truncated values | Field splitting is on whitespace | `awk -F'\t'` or `-F','` to match the real delimiter |

### 6. Comparison Table

| Task | GNU coreutils | Python one-liner | When to prefer which |
|------|---------------|-------------------|----------------------|
| Find lines | `grep -n pat f` | `python -c "..."` | grep: always (faster, clearer) |
| Replace text | `sed 's/a/b/g' f` | `re.sub` | sed for simple literal replaces; Python for regex with logic |
| Sum a column | `awk '{s+=$2} END{print s}' f` | `sum(map(float, ...))` | awk: no interpreter startup, streams huge files |
| Count occurrences | `grep -c pat f` | `.count()` | grep on files; Python on in-memory strings |
| Pretty JSON | `jq .` | `json.dumps(indent=2)` | jq: nested queries, filters, multiple files |
| Sort unique | `sort -u f` | `sorted(set(...))` | shell when piping into another tool |
| Heavy text logic | awk/sed | Python/Node | Python when the logic stops being a one-liner |

**Rule of thumb:** if the whole job fits in one readable pipeline, use the shell — it is composable with everything else. The moment you need a loop with state, switch to a real language.

---

## 🔗 Tech Cross-References
- **DSA** — judge I/O is exactly this: parse input, process, print output. `time ./a.out`, `diff <(./a.out < input.txt) expected.txt`, and `| less` on test output are the measurement loop in [`../dsa/01-learning-roadmap.md`](../dsa/01-learning-roadmap.md). Learning `grep`/`sort`/`uniq` on test output is the cheapest debugging habit in competitive programming.
- **Git** — `git log | grep -i fix`, `git diff | less`, `git log --stat | sort -k3 -n` are the standard ways to audit history. `git_github/10-workflows-cookbook.md` assumes this fluency.
- **Docker** — `docker ps | awk '{print $1}' | xargs docker logs -f`; CI logs are the same text, and `grep ERROR` is how you find the failing step.
- **YAML** — a config file is text, so it diffs and greps like any other: `grep -n "on:" .github/workflows/*.yml`. Wrong indentation becomes a visible line-number bug this way.
- **Databases** — `psql -c "..." | column -t`, and CSV exports (`psql -A -F,`) are pipeline stages; `grep` + `awk` over an export is a poor man's analytics layer.
- **Shell vs PowerShell** — Unix composes *text* with `|`; PowerShell composes *objects*. Same goal, different cost: on Unix every stage re-parses text, which is slower but far more flexible.
- **JavaScript** — string/array methods (`split`, `map`, `filter`, `reduce`) are the same operations as `cut`/`grep`/`awk`; recognising the mapping makes either tool easier to learn.

## 🧪 Practice Tasks
> No solutions provided. Do them, then ask AI to grade — or write a second, worse version first and compare.

- [ ] **Task 1 — Log forensics.** With a log file of ≥ 30 mixed lines, produce: (a) the 5 most frequent error messages with counts, (b) requests per minute, (c) the single slowest request. One pipeline per answer where possible; no loops in a language.
- [ ] **Task 2 — Field extraction.** Given `name:age:city` lines, use **only** `cut`, `awk`, and `sort` to output the average age and the count per city. Then redo it with a one-line Python equivalent and compare readability versus speed (`time`).
- [ ] **Task 3 — Sed safety drill.** Make three deliberate mistakes with `sed -i` on a copy of a config file (delete a line you shouldn't, over-replace a substring, break a regex), and recover each one using `.bak` or `git checkout`. Write down the habit you will build: *always `-i.bak` first*.
- [ ] **Task 4 — Shell → Python escalation.** Take a pipeline you built in Lab 1 that required `awk` gymnastics, and rewrite it in Python. Decide which version you would actually ship, and justify it in two sentences (readability, speed, or composability).
- [ ] **Task 5 — JSON with jq (PostgREST bridge).** Start a local database or use a public API returning JSON, fetch it with `curl`, and answer three questions with `jq` alone: count of records, list of one field, and the record with the highest numeric field. This is exactly how you will read data from PostgREST later.
- [ ] **Task 6 — Streaming vs buffering (Docker bridge).** Time `cat bigfile > /dev/null` versus `wc -l < bigfile` versus `grep pattern bigfile` on a large log. Explain the differences in terms of I/O and where each command's work happens.

---

## 📝 Research Journey
- **Why:** I could use `ls`, `cd`, and `git` — but whenever a tutorial said "pipe it into `awk`" I closed the tab. Every time I read a Stack Overflow answer with a six-stage pipeline, I could not tell which stage did what. This note exists so that a pipeline stops being magic and becomes something I can debug stage by stage.
- **Struggle:** The `2>&1` ordering rule. I knew the token, not the *reason*. It clicked only when I wrote both orders side by side in a lab and saw different results from identical-looking commands. Second struggle: `uniq` without `sort` — a bug that looks like `uniq` is broken when it is actually a contract mismatch.
- **Aha moment:** Redirections are evaluated **left to right**, and each stream (fd 0/1/2) is an independent pointer. Once I pictured "three pointers that redirect commands write to", `>`, `2>&1`, `|`, and `tee` all became the same idea applied at different points. And "everything is text" explained why any tool composes with any other tool.
- **Career link:** Debugging an AI system is almost always text archaeology: trace logs, model outputs, request/response JSON, container stdout, training metrics in CSV. `grep`/`awk`/`jq` fluency is the difference between "the pipeline is broken somewhere" and "step 4 of the ingestion job dropped 3% of the records". It is also the honest, reproducible way to measure an algorithm's behaviour — the measurement half of DSA.

## 🔗 Related Topics
- [[dev_foundation/linux/README.md]] — folder guide and roadmap
- [[dev_foundation/linux/01-file-system-paths-and-permissions.md]] — where the files you are now processing live
- [[git_github/03-core-daily-workflow.md]] — `git log | less` is the canonical pipeline
- [[dev_foundation/dsa/01-learning-roadmap.md]] — test output analysis during algorithm practice
- [[dev_foundation/databases/04-postgrest-auto-rest-api.md]] — `curl | jq` against a real API
- [[dev_foundation/docker/README.md]] — container logs are pipeline material
- [SUGGESTED] `04-bash-scripting.md` — when pipelines need conditionals and functions, the language has to grow; this note is the boundary
- [SUGGESTED] `bridge-text-processing-for-ml.md` — bridge note: `awk`/`jq`/`sed` on JSONL/CSV datasets, connecting to `ai_ml/` data pipelines

## 🤔 Open Questions
- [ ] When does GNU `parallel` (or `xargs -P`) become worth it — and what is the actual speedup on this machine?
- [ ] Should we look at `ripgrep` (`rg`) and `fd`, the modern replacements for `grep`/`find`? Faster, but less portable across servers.
- [ ] How much `jq` fluency is enough for reading PostgREST and LLM API responses?
- [ ] `set -euo pipefail` deserves its own section — which flags matter in practice?

---
*Unix gives you 20 small tools and one idea: connect them. Mastery is not memorising flags — it is being able to point at any stage and say what it does to the data.*

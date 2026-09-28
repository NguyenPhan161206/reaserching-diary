# Linux — Navigation & Core CLI

**Created Date:** 2026-09-28
**Last Updated:** 2026-09-28
**Tags:** `#Linux`, `#Shell`, `#CLI`, `#Navigation`, `#FileSearch`, `#ManPages`, `#DevTools`
**References:** `man 1 bash`, `man 1 find`, `man 1 getopt` (the convention every CLI follows), GNU coreutils manual, Bash Reference Manual §3 (expansion order)
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`01-file-system-paths-and-permissions.md`](01-file-system-paths-and-permissions.md)

---

## 📌 Key Takeaways (TL;DR)
- **The working directory is process state, not a location in the filesystem.** `cd` changes a variable inside one process; it writes nothing to disk and does not move you anywhere. This single fact explains why `cd` in a script "does not work", why `cd ~` in a subshell is gone afterwards, and why two terminals in the same folder are independent.
- **Before you run any command, the shell has already rewritten your line eight times.** Brace → tilde → parameter → command substitution → arithmetic → word splitting → **glob** → quote removal. Every surprising result in this note is one of those eight steps.
- **A quoted `*` is not a glob.** Quoting protects a character from expansion, which is why `'*'` and `"$HOME"` behave differently from `*` and `$HOME`.
- **Never parse `ls` output.** Filenames may contain spaces, newlines, and quotes, and `ls` quotes differently when it writes to a pipe than when it writes to a terminal. `find -print0 | xargs -0` is the only safe pipeline.
- **`type`, not `which`.** `type` is a shell builtin that sees aliases, functions, builtins, and `PATH`; `which` is a separate program that sees only the `PATH` search and lies about builtins and aliases.
- **`find` walks the live tree; `locate` queries a stale index.** `find` is always right and slow, `locate` is fast and wrong (a file created 30 seconds ago will not be in the database). Choose by whether you need *now* or *fast*.
- **`man` pages are sectioned**, and the section is part of the name: `man 1 ls` (command) vs `man 3 strlen` (C library call) vs `man 5 crontab` (file format). The man page you want is frequently not the one you get by default.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

#### 1.1 The shell has exactly three jobs

```
   you type:      ls -l notes/
        │
        ▼
   ┌────────────────────────┐   ① PARSE      read words, expand, quote,
   │  the shell            │                handle redirections
   └────────────────────────┘
        │
        ▼
   ┌────────────────────────┐   ② RESOLVE    is `ls` an alias? a function?
   │  the shell            │                a builtin? or a file in PATH?
   └────────────────────────┘                → start that program
        │
        ▼
   ┌────────────────────────┐   ③ EXECUTE    the program inherits fd 0/1/2
   │  the program          │                and runs; the shell WAITS
   └────────────────────────┘
        │
        ▼
   exit status  ──►  shell's `$?`
```

Everything in this note is a consequence of steps ① and ②. When a command "does not exist", the answer is almost always in ②; when a command "does something weird with my text", the answer is in ①.

#### 1.2 `cd` changes a variable, not a place

Your prompt like `user@host:~/notes/linux$` is a **rendering of a variable**. `cd` performs `chdir()` on the running shell process, updating one integer (the current working directory) inside that process's address space. Nothing is recorded on disk.

Three consequences you will meet within a week:

```bash
# 1. A subshell has its own copy → your cd is discarded
(cd /tmp && pwd)   # /tmp
pwd                # still where you were

# 2. Another terminal window is another process → unaffected by your cd
#    (only a *shared* directory on disk is shared)

# 3. A program started with a different working directory is unaffected:
#    (cd /etc && make)   # make runs in /etc, not in your project
```

**The practical fix for scripts** is the compound command, which runs both parts in *one* shell:

```bash
cd "$(dirname "$0")" && java -jar app.jar     # ✅ same shell, cd persists
(cd "$(dirname "$0")" && java -jar app.jar)    # ❌ subshell, cd discarded
```

#### 1.3 Path resolution: the search order for a command name

When you type `git`, the shell tries these **in order** and stops at the first hit:

| Order | Kind | Example | How to inspect |
|-------|------|---------|----------------|
| 1 | alias | `alias ll='ls -alF'` | `type git` |
| 2 | shell function | `git() { command git "$@"; }` | `type git` |
| 3 | shell keyword | `for`, `time`, `if` | `type for` |
| 4 | builtin | `cd`, `echo`, `export`, `read`, `type` | `type cd` |
| 5 | `$PATH` entry (first match wins) | `/usr/bin/git` | `command -v git` |
| 6 | **PATH order matters** | `/usr/local/bin` precedes `/usr/bin` on Ubuntu/Debian | `echo $PATH` |

```bash
$ type -a echo
echo is a shell builtin
echo is /usr/bin/echo
echo is /bin/echo
```

**Read that carefully:** a `$PATH` search would find `/usr/bin/echo`, but the builtin wins. (The last line appears twice because on Ubuntu `/bin` is a symlink to `/usr/bin` — a reminder that `$PATH` entries are searched in order and not deduplicated.) The same is true for `pwd`, `test`, and `time` — and for `cd`, which has **no** file on disk at all. This is why `which echo` (which only searches `$PATH`) is misleading.

#### 1.4 Filenames are opaque bytes

A file is named by a sequence of bytes that ends at a NUL. There is **no requirement** that it contain letters, no extension, and no limit besides 255 bytes per component. These are all legal:

```
report.pdf          2 files with a space      a file
with spaces         my (2) copies?           -rf
-rf                 café.txt                 file
                                          backslash-name
                                          $(whoami)      ← literal characters!
```

Consequences that matter:

- **Quotes in names break naive scripts.** `for f in $(ls)` splits on whitespace; `rm "$f"` handles one file with spaces, but the loop variable was already shattered.
- **A filename can look like a command.** `-- -rf` is a directory named `-rf`; a file named `$(whoami)` is *not* executed when you type `ls $(whoami)`, because the shell expands the *word you typed*, not the name it read from disk. (The danger appears the moment you build a command string and run it through `sh -c` or `eval` — do not.)
- **Newlines in filenames break line-oriented tools** for real. `find -print0` and `xargs -0` exist for exactly this.

#### 1.5 Three different meanings of "where"

| Question | Command | Layer |
|----------|---------|-------|
| Which directory is this shell in? | `pwd` | shell/process state |
| Where does that path really point on disk? | `readlink -f .` / `pwd -P` | filesystem, resolving symlinks |
| Which files are here? | `ls -A` | directory entries |

`pwd` prints the **logical** path (the one you typed, with symlinks intact); `pwd -P` prints the **physical** path (all symlinks resolved). They disagree inside a symlinked directory, and that disagreement is a real source of confusion when comparing a path in a config file with the path your shell shows.

---

### 2. Mechanism (How It Works)

#### 2.1 The eight expansions, in order

Before *any* command runs, bash rewrites your line. The order is fixed, and several classic bugs exist purely because of it:

| # | Expansion | Example input | Result |
|---|-----------|---------------|--------|
| 1 | brace | `cp a/{x,y}.txt .` | `cp a/x.txt a/y.txt .` |
| 2 | tilde | `cd ~` / `cd ~user` | `cd /home/me` |
| 3 | parameter | `echo $HOME/*.md` | the *value* of `$HOME` inserted |
| 4 | command substitution | `echo $(date +%F)` | output of the inner command |
| 5 | arithmetic | `echo $((1+2))` | `3` |
| 6 | word splitting | `v="a b"; rm $v` | `rm a b` ← **two arguments** |
| 7 | **pathname (glob)** | `ls *.md` | sorted list of matching filenames |
| 8 | quote removal | `echo '$HOME'` | literal `$HOME` |

The three rules that follow:

```bash
# Rule 1: word splitting happens BEFORE globbing.
v="notes linux"
echo $v/*.md        # 3 tokens → globs notes/*.md and linux/*.md
echo "$v"/*.md     # 1 token → tries to glob "notes linux"/*.md  → no match

# Rule 2: globbing is the LAST step that can add arguments, and quotes protect from it.
ls *               # globs
ls '*'             # prints the literal character *
ls ./*.md          # same as *.md here, but survives a leading-dash filename

# Rule 3: a glob that matches nothing is passed through as a literal string.
$ echo *.nomatch
*.nomatch
$ rm *.nomatch
rm: cannot remove '*.nomatch': No such file or directory
```

**Rule 3 explains the most confusing "the command says no such file" moment in Unix:** the shell did not fail to glob, it globbed to nothing and handed the pattern itself to the program. Two ways to make the failure loud instead:

```bash
shopt -s nullglob          # unmatching patterns vanish (→ "missing operand" error)
shopt -s failglob          # bash refuses: "no match"  ← prefer this
shopt -s dotglob nocaseglob globstar   # * matches .files, A == a, ** is recursive
```

#### 2.2 Tab completion: a shell feature, not a filesystem feature

Readline asks the shell for a list of candidates using rules the shell knows (from `$PATH`, `COMP_WORDBREAKS`, and completion functions), and inserts the **longest common prefix**. Press <kbd>Tab</kbd> twice to list all candidates.

What completion does that `ls` does not:

- inserts the prefix even when **several** candidates match, so you can see the ambiguity without typing the whole name;
- **escapes special characters** — a file named `my notes/` completes as `my\ notes/`, which is safe to pass as one argument;
- knows about flags (`--<Tab>` lists long options — often the fastest way to discover a tool's flags);
- can be *programmed* (completion functions in `/etc/bash_completion.d/` define how `git che<Tab>` becomes `git checkout`).

**Important caveat:** a trailing space is added when the completion is unambiguous. In a **script**, that silently splits one argument into two. Never let a script depend on interactive completion — and never paste a tab into a script.

#### 2.3 What `find` actually does

```
find STARTING_POINT [expression]
                     │
                     ├── read the directory entry of STARTING_POINT
                     ├── for each entry: evaluate the expression, left to right,
                     │   with an implicit -a (AND) between predicates
                     │        true → -print (default action)
                     │        false → skip this entry (children are NOT visited
                     │                only when you -prune)
                     └── recurse into subdirectories (depth-first, readdir order)
```

Three mechanisms explain nearly all `find` mistakes:

1. **Predicates are an AND-chain, evaluated left to right.** `-print` is an *action* that returns `true`; once it runs, the rest of the chain is short-circuited:

   ```bash
   find . -name '*.log' -print     # ✅ prints only .log files
   find . -print -name '*.log'     # ❌ prints EVERYTHING (name is never reached)
   ```

2. **`-prune` stops the descent, so it must be combined with `-o` when the pattern should also match directories:**

   ```bash
   # the canonical "skip noisy directories" idiom
   find . \( -name node_modules -o -name .git \) -prune -o -type f -name '*.ts' -print
   ```

3. **`-exec` comes in two flavours.** `\;` runs one process *per match*; `{} +` batches many arguments into one process:

   ```bash
   find . -name '*.log' -exec rm {} \;    # n processes, n forks
   find . -name '*.log' -exec rm {} +    # ceil(n/ARG_MAX) processes
   ```

   Both forms pass the filename to the child process as a plain **argument** — nothing is re-parsed by a shell, so a file literally called `; rm -rf ~` is just a very strange argument, not a second command. The thing `-exec` cannot save you from is a *tool* that then reinterprets its arguments (see below); the thing it *does* cost you is one `fork` per match with `\;`. For untrusted filenames the safe form is still `-print0` + `xargs -0`, but the reason is whitespace and newlines, not command injection:

   ```bash
   find . -name '*.log' -print0 | xargs -0 grep -l ERROR   # -0 means "NUL-separated, no quote processing"
   find . -name '*.log' -exec grep -l ERROR {} +           # same, with fewer processes
   ```

   Real injection needs a shell in the middle, and that is on you: `-exec sh -c 'rm "$1"' _ {} \;` re-introduces every quoting problem, and there is no flag that makes it safe.

#### 2.4 `find` vs `locate` — the only real difference that matters

| | `find` | `locate` |
|---|---|---|
| Data source | **walks the live tree** (`readdir` per directory) | **queries a prebuilt database** (`updatedb`, usually daily) |
| Accuracy | always correct for the current path | index is stale: new files missing, deleted files still listed |
| Speed | $O(\text{entries visited})$ — slow on `/` | $O(1)$ lookup per name — instant |
| Needs permission | needs read on directories | returns names you may not be allowed to open |
| Answer to "does this file exist *right now*?" | yes | **no** |

A third option, `ripgrep`/`fd`, is a *content*-oriented search with sane defaults (skips `.gitignore`d files, walks in parallel). `fd` is a modern `find` replacement; `rg` is a modern `grep`. They are dramatically nicer on a code base and unavailable on a minimal server — so learn the standard tools first and treat the modern ones as upgrades.

#### 2.5 Finding the right man page

`man` pages are organised in **sections**, and the section number is part of the name:

| Section | Contents | Example |
|---------|----------|---------|
| 1 | user commands | `man 1 ls`, `man 1 git` |
| 2 | system calls | `man 2 open`, `man 2 mmap` |
| 3 | C library functions | `man 3 strlen`, `man 3 printf` |
| 4 | devices & filesystems | `man 4 pts` |
| 5 | file format | `man 5 fstab`, `man 5 passwd`, `man 5 crontab` |
| 6 | games | — |
| 7 | misc / troff | `man 7 regex`, `man 7 inode` |
| 8 | admin commands | `man 8 systemd`, `man 8 crontab` |

```bash
man 1 ls            # the command
man 3 strlen       # the C function — a DIFFERENT man page with no `ls` mention
man -w ls          # just the path of the man page (is it even installed?)
man -k inode       # search page titles/descriptions  (same as `apropos inode`)
man -S 2 open      # list every page mentioning "open" in section 2
man bash           # the shell itself — 6000+ lines, the single most useful man page
help cd            # builtins have no man page on most systems; use `help`
```

**The section-number trap, with a real example.** Two of the most-wanted pages share a name with a command:

```bash
$ man 1 passwd | head -1
PASSWD(1)     User Commands        PASSWD(1)     # "change your password"

$ man 5 passwd | head -1
PASSWD(5)     File Formats Manual   PASSWD(5)     # the format of /etc/passwd

$ man crontab | head -1              # default lookup → the COMMAND (install a crontab)
$ man 5 crontab | head -1            # the FILE FORMAT you actually wanted
```

**Reading a man page efficiently:** skip to `SYNOPSIS` (does it match my invocation?), then `DESCRIPTION`, then the specific `OPTION` you care about, then `EXAMPLES`/`SEE ALSO`. The `NAME` line tells you the section and the one-line purpose.

Every serious CLI also supports `--help`, and it is often *better* than the man page because it is short, version-matched to the binary on your disk, and grouped by task. Rule: `--help` first, `man` second, `info` third (GNU `info` pages for coreutils contain the tutorials the man page omits), source code last.

---

### 3. Command Cheat Sheet

#### 3.1 Where am I / what is here

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `pwd` | Print working directory | Confirm where you actually are | `/home/me/reaserching-diary` |
| `pwd -P` | Physical path (symlinks resolved) | Compare against a path in a config/log | `/home/me` |
| `ls -A` | List, **including** dotfiles, excluding `.`/`..` | Default "what's here" | `notes logs "my notes"` |
| `ls -lah` | Long + human sizes + all | The only `ls` worth aliasing | `total 24K ...` |
| `ls -lt` | Sorted by mtime, newest first | "What changed recently?" | — |
| `ls -lS` | Sorted by size, largest first | Spot the file that ate the disk | — |
| `ls -li` | Long + inode number | Detect hardlinks / same file twice | — |
| `ls -d */` | Directories only | Quick orientation | `ai_ml/ git_github/` |
| `ls -1` | One name per line | Readable in a script-ish context | — |
| `tree -L 2 -d` | Tree, depth-limited, dirs only | Structure at a glance (may need `sudo apt install tree`) | — |
| `cd ..` | Parent directory | Universal | — |
| `cd -` | Previous directory (`$OLDPWD`) | Toggle between two spots | — |
| `cd` (no args) | `$HOME` | Reset | `/home/me` |
| `cd ~user` | Another user's home | Grab a config | — |
| `cd -- -weird` | A directory literally named `-weird` | Escape leading-dash names | — |
| `pushd d/ ; popd` | Directory stack | Zoom in and back out of nested dirs | — |
| `readlink -f path` | Absolute, symlink-resolved path | Feed a tool a path it cannot resolve itself | `/home/me/notes` |
| `realpath --relative-to=a b` | Path of `b` relative to `a` | Build paths for configs | `../notes` |

#### 3.2 Create / move / delete

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `mkdir -p a/b/c` | Create tree, no error if exists | Scaffolding; scripts | silent |
| `touch f` | Create empty / update mtime | Marker file, or "I was here" before a crash | — |
| `cp src dst` | Copy a file | — | — |
| `cp -a src/ dst/` | Archive: recursive, preserve mode/times/links | Copying a whole tree safely | — |
| `cp -r src dst` | Recursive (does not preserve everything) | Quick copy of a tree | — |
| `mv src dst` | Move / rename | **Atomic if same filesystem**; a copy+delete across filesystems | — |
| `rm file` | Delete one file | — | — |
| `rm -i f1 f2` | Prompt before each | Deleting anything you might regret | `rm: remove 'f1'?` |
| `rm -r dir/` | Delete a tree | Only after `ls` the contents | — |
| `rm -rf dir/` | No prompts, recursive | **Gotos** — see Lab 4 for the safe version | — |
| `rmdir empty_dir/` | Delete an empty dir | Fails loudly if not empty — a safety feature | — |
| `gio trash f` | Move to the desktop trash | Recovery window; Ubuntu ships `gio` | recoverable |
| `ln -s target linkname` | Symbolic link | Pin a config, alias a path | — |
| `stat -c '%A %U %G %s %y %i' f` | One-line metadata | Answer "who owns it, how big, when, which inode" | `-rw-r--r-- me me 1204 ...` |
| `file f` | Detect type by magic bytes | "It has no extension — what is it?" | `ASCII text` |

#### 3.3 Search

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `ls *.md` | Shell glob | One level, fast, already sorted | — |
| `ls **/*.md` | **Recursive glob** | Only with `shopt -s globstar`, else same as `*` | — |
| `find . -name '*.log'` | Recursive by name | Canonical filesystem search | — |
| `find . -iname '*.log'` | Case-insensitive | macOS files, user input | — |
| `find . -type d -name node_modules -prune -o -type f -print` | Prune noise dirs | **The** code-base search | — |
| `find . -type f -mmin -60` | Modified in the last 60 minutes | "What did I just touch?" | — |
| `find . -type f -newermt '2026-09-28'` | Modified after a date | Recover from "what was I doing" | — |
| `find . -type f -size +500c` | Larger than 500 bytes (`c`=bytes exact) | Find the heavy files | — |
| `find . -type f -empty` | Empty files | Find dead config / broken volume mounts | — |
| `find . -name '* *' -print` | Names **containing a space** | Audit quoting problems | `"./my notes"` |
| `find / -xdev -name x -type f 2>/dev/null` | Whole-filesystem search, one device | Finding a binary — `-xdev` stops at mounts | — |
| `find . -name '*.log' -print0 \| xargs -0 grep -l ERROR` | Safe batched content search | Filenames with spaces/newlines | — |
| `locate file` | Query the file index | "Which config is that on this server?" | ⚠️ possibly stale |
| `updatedb` | Rebuild the `locate` index (needs root) | After creating files you must locate | — |
| `fd pattern` | Modern `find` (regex, respects ignore files) | Fast code-base navigation | — |
| `rg pattern` | Modern recursive `grep` (respects `.gitignore`) | Searching a code base | — |
| `grep -rn "TODO" .` | Recursive content search | Portable version of `rg` | — |
| `grep -c "" f` | Count lines without `wc` | — | `1204` |

#### 3.4 Read documentation and find commands

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `cmd --help` | Built-in usage summary | **First stop**, always version-matched | usage block |
| `man cmd` | Full manual page | Need details, edge cases, POSIX mode | pager |
| `man N cmd` | Manual page of a specific section | `man 5 crontab`, `man 3 printf` | — |
| `man -k topic` / `apropos topic` | Search all pages | You know the *concept*, not the command | `file (5)` |
| `man -w ls` | Path of the man page | Is the package installed at all? | `/usr/share/man/man1/ls.1.gz` |
| `man -S 2 open` | Restrict search to a section | Disambiguate common names | — |
| `help cd` | Builtin help (no man page exists) | Shell builtins | `cd: cd [-L\|[-P [-e]] [-@]] [dir]` |
| `type cmd` | Show what the shell will run | Debug "which ls am I getting?" | `ls is /usr/bin/ls` |
| `type -a cmd` | Show **all** candidates in order | Alias shadowing a binary | 2+ lines |
| `command -v cmd` | The resolved path, or nothing | Script-safety check | `/usr/bin/ls` |
| `which cmd` | `$PATH` search only (**misleading**) | Legacy habit — prefer `command -v` | `/usr/bin/ls` |
| `compgen -c git` | List every command starting with `git` | Discover a tool you half-remember | list |
| `type -a python3` | Check interpreter shims | Version managers put one Python first in `$PATH` | 2 lines (`/usr/bin`, `/bin`) |
| `info coreutils 'ls invocation'` | GNU tutorial page | Deeper than the man page | — |

#### 3.5 History, state, and the shell's memory

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `history \| tail` | Recent commands | Recall an incantation | — |
| `Ctrl-R` | Reverse search | The single most useful shortcut in a shell | — |
| `!!` | Re-run the last command | Repeat when a flag was missing | — |
| `!$` | Last argument of the last command | `mkdir -p a/b/c; cd !$` | `a/b/c` |
| `!cmd` | Most recent command starting with `cmd` | `!git` | — |
| `fc -l -20` | List last 20 | Review before a destructive step | — |
| `set -x` | Echo each command before running | **Debug a script** — shows the actual expansions | trace |
| `set +x` / `set -v` | Stop / show unexpanded input | — | — |
| `export FOO=bar` | Put a variable in the environment | Config consumed by a program | — |
| `env \| sort` | Full environment | Find where a secret came from | — |
| `history -c` | Clear history | **Before typing anything secret** | — |
| `unset FOO` | Remove a variable | Same session | — |

---

### 4. Practical Lab

> Work in `/tmp/nav-lab`. Every command has a verifiable expected output — do not continue until yours matches.

#### Lab 1 — Build a tree and navigate it without the mouse

```bash
cd /tmp && rm -rf nav-lab && mkdir -p nav-lab && cd nav-lab
mkdir -p notes/dsa notes/linux archive/backup logs "my notes"
printf '# roadmap\n'          > notes/dsa/01-roadmap.md
printf '# java\n'             > notes/dsa/02-java.md
printf '# filesystem\n'       > notes/linux/01-fs.md
printf '# navigation\n'       > notes/linux/02-nav.md
printf '# inbox\n'            > notes/inbox.md
printf '# old\n'              > archive/old-note.md
printf '# backup\n'           > archive/backup/.keep
printf 'INFO start\n'         > logs/app.log
printf 'ERROR boom\n'         > logs/app.log
printf 'DEBUG noisy\n'        > logs/debug.log
printf 'buy milk\n'           > "my notes/todo.txt"
head -c 2000 /dev/urandom      > logs/big-blob.bin
```

```bash
# 1. Where am I?  (you are in /tmp/nav-lab)
pwd

# 2. Walk down and back up — the parent operator is a filesystem link
cd notes/linux && pwd        # /tmp/nav-lab/notes/linux
cd ../dsa && pwd             # /tmp/nav-lab/notes/dsa
cd -                          # back to .../notes/linux
echo $OLDPWD                  # /tmp/nav-lab/notes/dsa  ← "previous" is tracked state

# 3. Directory stack: descend twice, return without remembering the path
pushd ../../logs && pwd      # /tmp/nav-lab/logs
pushd ../archive/backup && pwd
popd && pwd                  # back down the stack, one level
popd                         # ... and to /tmp/nav-lab

# 4. What is here — including dotfiles, excluding . and ..
ls -A
```

**Expected output (step 4) — written to a terminal, so `ls` *quotes* the name with a space:**

```
archive   logs   'my notes'   notes
```

Alphabetical (GNU `ls` sorts unless told otherwise), sorted with the quoted entry inline. Now the same listing through a pipe — into a file, and byte-visible:

```bash
ls -A > listed.txt ; cat listed.txt        # quotes are GONE
ls -A | cat -A | grep notes                # cat -A: $ = end of line, non-printables escaped
```

```
archive
listed.txt
logs
my notes
notes
```

```
my notes$
notes$
```

`cat -A` adds the `$` end-of-line marker — and there is **no quote character anywhere**, because `ls` only quotes for a terminal. A filename containing a literal newline would show up here as a line that ends in `$` and then continues, which is the whole reason this output is unparseable.

**Why this matters:** a script that does `for d in $(ls -A); do ...; done` will iterate over `my` and `notes` separately and then fail on both. `for d in */` is the correct idiom for directories. Quoting is a *presentation* decision `ls` makes for a terminal — never parse it.

```bash
# 5. Resolve paths instead of guessing them
cd notes/linux         # the rest of this lab runs from here
readlink -f 02-nav.md  # → /tmp/nav-lab/notes/linux/02-nav.md
readlink -f ../../logs # → /tmp/nav-lab/logs, resolved even from a subdirectory

# 6. Metadata in one line
stat -c '%A %U %G %s bytes  inode=%i' 01-fs.md
```

```
-rw-rw-r-- binhnguyen binhnguyen 13 bytes  inode=3062016
```

(`13` is `printf '# filesystem\n'` — 12 characters plus the newline. The inode number is yours, not mine.)

```bash
file 01-fs.md ../../logs/big-blob.bin
```

```
01-fs.md:                ASCII text
../../logs/big-blob.bin: data
```

`file` pads the names into a column so the descriptions line up — the same presentation-only decision `ls` made. The 2 000 random bytes are `data` because no magic number leads them, which is also why `file` is a *hint*, not a validator.

**Note the permission bits:** they come from your **umask** (`umask` shows it), not from the command — the same `printf` gives `-rw-r--r--` on one machine and `-rw-rw-r--` on another. That is note 01's lesson in one line.

#### Lab 2 — The documentation drill

Pick six commands you use but cannot fully explain. For each, produce **four** facts.

```bash
# 1. What will the shell actually run?
type -a ls
type -a cd          # cd is a shell builtin — there is no /bin/cd
type -a echo        # builtin first, /usr/bin/echo second, /bin/echo third (/bin → /usr/bin)
command -v ls       # the resolved path (script-safe)

# 2. Find the man page and its section
man -w ls           # /usr/share/man/man1/ls.1.gz     ← section 1
man -w passwd       # /usr/share/man/man1/passwd.1.gz  ← the COMMAND, not the file format
man 5 passwd | head -3        # PASSWD(5) File Formats Manual ← the one you usually want
man -k inode | head -5        # search by concept, not by name → inode(7)
man -S 2 open | head -4       # only system calls mentioning "open" → open(2)

# 3. The fast path
ls --help | grep -A1 -- '-A,'       # discover flags without leaving the terminal
git --help                           # the three usage blocks
docker --help | head -20

# 4. Builtins have no man page
help cd
```

**Expected highlights:**

```
$ type -a cd
cd is a shell builtin

$ type -a echo
echo is a shell builtin
echo is /usr/bin/echo
echo is /bin/echo

$ which cd
                                   ← no output: `which` cannot see builtins. This is why
                                     `which` is the wrong tool.
```

**Checkpoint:** for each of the six commands, write down its manual section number and one flag you did not know before. If you cannot name the section, you do not know what kind of thing you are running.

#### Lab 3 — The search drill (and the staleness lesson)

```bash
cd /tmp/nav-lab

# 1. By name
find . -name '*.md'                       # 5 markdown files
find . -name '*.md' | wc -l
find . -name '*.log' -o -name '*.bin'     # OR needs -o, and beware precedence
find . -type d -empty                     # no output — archive/backup holds .keep

# 2. The two bugs that make find look broken
find . -print -name '*.log'                # ❌ prints EVERYTHING (see Lab 4 note below)
find . -name '*.log' -print                # ✅ prints only the .log files

# 3. By time  (careful: -mtime rounds DOWN, so -mtime -1 means "younger than 2 days")
find . -type f -mmin -60                  # touched in the last hour  ← precise
find . -type f -mtime -1                  # ⚠️ actually matches up to 48h old
find . -type f -newermt '2026-09-28 00:00'

# 4. By size (c = bytes, no block rounding — the honest unit)
find . -type f -size +500c                # only big-blob.bin

# 5. Prune the noise — the code-base idiom
find . \( -name .git -o -name node_modules \) -prune -o -type f -name '*.md' -print

# 6. Unsafe names: find one that would break a naive script
find . -name '* *'                        # → ./my notes

# 7. locate is a different machine: a prebuilt index
locate big-blob.bin                       # ❌ "nothing found" — the index predates the file
sudo updatedb && locate big-blob.bin      # ✅ now it works (and is still stale tomorrow)
```

**The lesson to write down:** `find` answers *"what exists now"*; `locate` answers *"what existed when the index was last built"*. During an incident, only the first question is worth asking.

```bash
# 8. Safe batching (filenames with spaces/newlines cannot break it)
find . -name '*.log' -print0 | xargs -0 grep -c ERROR
find . -type f -name '*.md' -exec wc -l {} +     # {} + = one wc for many files
find . -type f -name '*.md' -exec wc -l {} \;    # \; = one wc per file
```

#### Lab 4 — Breaking things on purpose, and recovering

```bash
cd /tmp/nav-lab

# ── The classic data-loss bug ─────────────────────────────────────────────
# Measured on GNU coreutils: `rm -rf <unquoted-empty>/ ` and even
# `rm -rf -- "<empty>/"` both reduce to `rm -rf /`, because the quoted
# empty string concatenated with "/" *is* the root path.
EMPTY=""

rm -rf $EMPTY/                 # ⚠️ unquoted + empty → rm -rf /
                                 #    GNU rm refuses ("dangerous to operate
                                 #    recursively on '/'") — by luck, not design

rm -rf -- "$EMPTY"             # ✅ becomes rm -rf -- '' → silent no-op (-f ignores
                                 #    missing operands). Safe, but also silent: you
                                 #    learn nothing, which is its own problem.

rm -rf -- "${EMPTY:?refusing to run with an empty path}"    # ✅ aborts LOUDLY
                                 #    bash: EMPTY: refusing to run with an empty path
                                 #    (exit 127) — the correct habit for scripts

# The rule: quoting protects against splitting, NOT against concatenation.
# `"$EMPTY"/` is not safe, and the difference is invisible in the source.

# ── Glob that matches nothing ─────────────────────────────────────────────
echo *.nomatch                 # → prints the literal string *.nomatch
shopt -s failglob
echo *.nomatch                 # → bash: no match
shopt -u failglob

# ── cd that "does not work" ───────────────────────────────────────────────
(cd /tmp && ls)                # subshell: cd discarded
cd /tmp && ls; cd -            # ✅ compound: same shell, cd persists

# ── three real mistakes, three recoveries ─────────────────────────────────
mkdir a/b/c                    # ❌ a/b does not exist
mkdir -p a/b/c                 # ✅
rm -rf notes                   # ❌ irreversible — there is no "undo" for a recursive delete
gio trash notes                # ✅ the recovery you actually have
rm a/b                         # ❌ "Is a directory"
rm -r a/b                      # ✅

# ── The safe-delete default ───────────────────────────────────────────────
gio trash a/b                  # recoverable; then empty the trash manually
```

**There is no recovery for `rm -rf` that does not have to have happened first.** `git checkout -- .` restores *tracked* files only — every untracked note you just made is gone, and in a scratch directory like this one there is no repo at all. Your options, in order of how much they cost you:

| Recovery | Restores what | Cost |
|---|---|---|
| `gio trash <path>` / `trash-put` | Anything, tracked or not | Files move to a trash directory you still have to empty yourself |
| `cp -a` (or `tar`) before deleting | Anything, on a schedule | One extra copy per destructive command |
| `git checkout -- .` | **Tracked files only** | Free, and useless for exactly the files you just typed |
| The trash itself | — | `gio trash --empty` is permanent |

That table is the actual argument for `gio trash`: the recovery has to be set up *before* the mistake, so make it the default and stop thinking about it.

**The habit to build:** before any recursive delete, type `ls` (or `find . -maxdepth 1`) in the same command line — and never let a variable that may be empty reach `rm -rf` unquoted or unvalidated.

---

### 5. Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---------|---------|-----|-----|
| `rm -rf $DIR/` with `DIR` empty | Attempts to delete `/` | Unquoted empty variable expands to *nothing*, so the command becomes `rm -rf /` | `rm -rf -- "${DIR:?message}"` — aborts if unset/empty |
| `rm -rf "$DIR"/` (quoted, still empty) | **Still** `rm -rf /` | Quoting stops *splitting*, not *concatenation*: `""` + `/` = `/` | Quote the whole thing (`"$DIR"`), and prefer the `${DIR:?}` guard |
| `rm -rf` with a mistyped variable (`$SRC` intended, `$SRCC` typed) | Deletes the wrong tree | Nothing validates the name | `set -u` (error on unset variables) + print the target before deleting |
| `for f in $(ls)` | Loop variable splits; files "not found" | `ls` output is re-split on whitespace; quoting differs when piped | `for f in */` or `for f in *` (unquoted glob) |
| `find . -print -name '*.log'` | Prints every file in the tree | Predicates are ANDed left to right; `-print` short-circuits | Put the test **before** the action |
| `find . -name 'x' -o -name 'y' -print` | Some matches not printed | Without parentheses, `-print` binds only to the second branch | `find . \( -name x -o -name y \) -print` |
| `find /` on a machine with mounts | Runs for minutes, hits `/proc`, `/sys` | Crosses filesystems and virtual trees | `find / -xdev 2>/dev/null` |
| `-mtime -1` | Matches files up to 48h old | Age in days is **truncated** | `-mmin -60` or `-newermt` |
| `find -size +1M` "finds" a 1.1 MB file unexpectedly | Sizes look wrong | `-size` rounds **up** to the next unit | Use `c` (bytes) for exact values |
| `locate newfile` | Not found; deleted files still listed | Index is a snapshot, updated daily | `find` for live truth, `updatedb` if you must |
| `cd` in a script / subshell "does not work" | Directory unchanged afterwards | `cd` modifies one process; subshells get a copy | `cd dir && cmd` (no parentheses) |
| `cd notes; ls` on one line with `;` then finding the wrong dir | Wrong output | `;` continues even if `cd` failed (`&&` stops) | Use `&&`, or `cd dir \|\| exit 1` in scripts |
| Parsing `which` | `which cd` prints nothing, or points at a shim | `which` only searches `$PATH`; it cannot see builtins/aliases/functions | `type -a` (interactive), `command -v` (script) |
| An alias silently missing in a script | "command not found" for something that works by hand | Aliases are **not** expanded in non-interactive shells | Use the full command, or `shopt -s expand_aliases` |
| Tab completion in a script | One argument becomes two | Completion inserts a trailing space when unambiguous | Never rely on completion in scripts |
| `ls huge_dir/*` | `Argument list too long` | The shell builds one huge argv | `find … -exec … {} +` or a loop |
| Unquoted `$VAR` anywhere | "too many arguments" / wrong file / glob expansion of a value | Word splitting + globbing on unquoted expansions | Quote every expansion: `"$VAR"` |
| `sudo somecommand` "works" at a terminal, fails in CI | `command not found` under `sudo` | `sudo` resets `PATH` via `secure_path` | Use the absolute path, or `sudo -E` with care |
| `cd /some/-weird-dir` | Treated as an option | Leading `-` in an argument | `cd -- -weird-dir`, or `cd ./-weird-dir` |
| Pasting a command that contains a tab | Argument split in odd places | Copy/paste keeps the literal tab | Strip tabs; use spaces |
| `history` contains a password/token | Secret recoverable from the shell history file | Default bash history records everything | `export HISTCONTROL=ignorespace` and start secrets with a space; `history -c`; prefer an env var over a CLI flag |
| `cd` into a directory you cannot traverse | `Permission denied` (note 01) | `+x` on the directory is required, `+r` only lists names | `sudo` or fix the mode — see [`01-file-system-paths-and-permissions.md`](01-file-system-paths-and-permissions.md) |

### 6. Comparison Table

#### 6.1 Which search tool, when

| Tool | Searches | Live or indexed | Respects `.gitignore` | Use it when |
|------|----------|-----------------|----------------------|--------------|
| shell glob `*`, `?`, `[a-z]` | names, one level, sorted | live (readdir) | no | You know the depth; fastest for typing |
| `find` | names + metadata (time, size, type, owner) | live, walks the tree | no | You need accuracy or a metadata predicate |
| `locate` | names, whole system | **indexed, stale** | no | "Which of the 40 copies of this config is in use?" |
| `fd` | names | live, parallel | yes | Modern, human-friendly `find` (needs install) |
| `rg` | **content** + names | live, parallel | yes | Searching code for a symbol or string |
| `grep -rn` | content | live, single-threaded | no | The portable version of `rg` (always available) |

#### 6.2 Which documentation source, when

| Source | Best for | Cost | Version accuracy |
|--------|----------|------|------------------|
| `cmd --help` | "what flags exist?" | seconds | exact (the binary on your disk) |
| `man cmd` | semantics, edge cases, POSIX mode | a minute | versioned, sometimes older than the binary |
| `man -k` / `apropos` | finding a command from a *concept* | seconds | n/a |
| `info coreutils` | the tutorials the man page omits | minutes | GNU only |
| `man N` (sections 2/3/5) | syscalls, C library, file formats | minutes | n/a |
| `tldr` pages | 80% use case in 30 seconds | seconds | community-written, may be stale |
| source code | the only truth for `-O2`/vendored forks | hours | exact |

**Rule:** `--help` to find the flag, `man` to understand the flag, source to explain the flag's behaviour that the docs get wrong.

#### 6.3 Shell vs PowerShell equivalents

| Task | Bash | PowerShell |
|------|------|------------|
| List directory | `ls -A` | `Get-ChildItem -Force` |
| What will run | `type -a` | `Get-Command` |
| Move | `cd ..` / `cd -` | `cd ..` / `cd -` |
| Filter a listing | `ls *.md` | `Get-ChildItem *.md` |
| Search content | `grep -rn` | `Select-String` |
| Pipeline data type | **text** | **.NET objects** |
| Wildcard character | `*` | `%` in some cmdlets, `-Filter` |
| Script default shell | `bash` | `powershell.exe`/`pwsh` |

The one conceptual difference worth internalising: in bash a pipeline passes **strings**, so each stage re-parses text; in PowerShell it passes **typed objects**, so `Select-String` can filter on properties without regex gymnastics — and a pipeline that fails tells you less about where the data changed, because you cannot `cat` it.

---

## 🔗 Tech Cross-References
- **Git** — every object Git stores is a file, so navigation is the Git CLI's job: `git ls-files` is a *better* `ls` for a repo (it lists tracked files, not build output), and `git rev-parse --show-toplevel` finds the repo root without walking up by hand. `find` + `.gitignore`-unaware is exactly why `rg` is preferred in a code base → [`../../git_github/01-version-control-concepts.md`](../../git_github/01-version-control-concepts.md)
- **Permissions** — you need `+r` to *list* a directory and `+x` to *enter* it. `cd` failing with `Permission denied` while `ls` works is the gap between those two bits → [`01-file-system-paths-and-permissions.md`](01-file-system-paths-and-permissions.md)
- **Docker** — the container has its own filesystem, its own `PATH`, and its own `pwd`. `docker exec -it web sh`, `docker cp app:/etc/config .`, and `cat /etc/os-release | grep PRETTY` are the same tools applied to a different namespace → [`../docker/README.md`](../docker/README.md)
- **YAML** — config discovery is a navigation problem: `find . -name '*.yml'`, `less .github/workflows/ci.yml`, `grep -n 'on:' .github/workflows/*.yml`. Indentation bugs become visible the moment you read the file with line numbers → [`../yaml_config/README.md`](../yaml_config/README.md)
- **Shell vs PowerShell** — see §6.3. Same goal, different data model for the pipeline.
- **DSA** — the judge loop is navigation plus redirection: `find . -name 'in*.txt' | sort`, `./a.out < input.txt | diff - expected.txt`, and `g++` reporting errors you then read with `grep -A3 error`. Reading compiler output with CLI tools is half of debugging algorithms → [`../dsa/01-learning-roadmap.md`](../dsa/01-learning-roadmap.md)
- **Databases** — `psql -c '\dt'` inside a `docker exec` shell is the same navigation model: a client is just a terminal program that lives in a directory and reads a `PATH` → [`../databases/README.md`](../databases/README.md)

## 🧪 Practice Tasks
> No solutions provided. Do them first, then ask AI to grade — or ask for a harder variant, a "why is this wrong" review, or a timing comparison.

- [ ] **Task 1 — Become faster than the mouse.** For one full day, handle *all* file navigation with the keyboard: no mouse-driven file manager, no dragging. Record the three commands that saved you the most time. Then deliberately repeat a 5-step "open a file, find a symbol, edit, save" workflow and time it before/after.
- [ ] **Task 2 — The `find` gauntlet.** In this repository (or any project you have locally), produce: (a) the 10 largest files under 5 MB, (b) every file modified in the last 24 hours, (c) every file whose name contains a space or a capital letter, (d) a count of files per top-level directory. One command per answer; `find` only.
- [ ] **Task 3 — Build a documentation index.** Choose 12 commands from your daily use. For each, record: manual section, one non-obvious flag with a one-line description, and the source (`--help`, `man`, `info`, or a URL). Present it as a 12-row table. Re-check one entry per week for a month.
- [ ] **Task 4 — Break and recover, on purpose, in a scratch directory.** Create a directory, then reproduce *all* of these: unquoted empty variable in `rm -rf`, glob with no match, `(cd x)` subshell discard, `find` predicate-order mistake, a filename with a space and one with a newline, and a leading-dash directory. For each: the exact command, the exact error message, and the fix. Keep this list; it is your own pitfall reference.
- [ ] **Task 5 — Prove the expansion order.** Write one command that fails *only* because of rule 5 (word splitting before globbing), and another that fails *only* because of rule 7→8 (quote removal after globbing). Explain each failure in two sentences.
- [ ] **Task 6 — `man` scavenger hunt (harder).** Find the manual page for: how `/etc/passwd` is parsed; the syscall that lists directory entries; the format of crontab; how the shell resolves `$PATH`. Record the section number and the paragraph that answered the question. Then answer: why is there no `man 5 ls`?
- [ ] **Task 7 — Script-grade navigation.** Write a shell script `whatis.sh` that takes one argument and prints, in order: resolved path (`command -v`), kind (`type`), manual page path (`man -w`), and the first usage line (`--help | head -1`). It must work for both `cd` and `ls`, and must not break when the argument is not a command.
- [ ] **Task 8 — DSA bridge.** Write a `judge.sh` that, given a problem directory, finds every `in*.txt`/`out*.txt` pair, runs `./a.out < in > got`, and prints a pass/fail table using `diff` and exit codes. Then measure one solution's scaling with `time` at $n = 10^4, 10^5, 10^6$ and infer the exponent from the ratios — this is the first half of the complexity toolkit ([`../dsa/04-complexity-analysis-toolkit.md`](../dsa/04-complexity-analysis-toolkit.md)).

---

## 📝 Research Journey
- **Why:** I could type `ls` and `cd`, but my navigation was mouse-driven: File Explorer, IDE tree view, dragging a folder. That worked until I was on a server, inside a container, or reading a stack trace that named a file I had never opened. There is no mouse on a CI runner. This note is about becoming fast with the tools that always exist.
- **Struggle:** Two things, both invisible from documentation. First, `cd` in a script "not working" — I had memorised `cd` as *moving*, not as *assigning*, so a subshell made no sense. Second, `ls` quoting differently when piped: I lost an hour to a loop over `$(ls)` before I understood that the terminal and a pipe are different destinations. The lab where I ran `ls -A | cat -A` and compared bytes fixed both permanently.
- **Aha moment:** The eight expansions, in a fixed order, explained every "the shell did something weird" moment I had collected for months — unquoted variables splitting into multiple arguments, `'*'` not globbing, a glob passing through as a literal when it matched nothing. Once the line is *rewritten* before the program ever sees it, the program is not misbehaving; it received different input than I intended. A second, smaller aha: `type -a echo` printing a builtin and a binary side by side killed my trust in `which` permanently.
- **Career link:** The first ten minutes of any incident are navigation: find the log, find the config, find the mounted volume, find the process. ML and MLOps make this worse, not better — a training job's real state is spread across `docker logs`, a config YAML, a CSV on a volume, and a metrics file, and nobody hands you the paths. `find`/`stat`/`readlink` fluency is unglamorous and decides whether an incident is a five-minute or a five-hour job.

## 🔗 Related Topics
- [[dev_foundation/linux/README.md]] — folder guide and roadmap
- [[dev_foundation/linux/01-file-system-paths-and-permissions.md]] — what the paths you are typing actually resolve to
- [[dev_foundation/linux/03-text-processing-and-pipes.md]] — once you are in the right place, this is how you read what is there
- [[git_github/01-version-control-concepts.md]] — `git ls-files` and the repo root, Git's version of navigation
- [[dev_foundation/databases/README.md]] — `psql` and `mysql` are just programs found in `$PATH`
- [[dev_foundation/dsa/01-learning-roadmap.md]] — Phase 0 tooling: compile, run, read the error, all from the terminal
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `06-bash-scripting.md` — loops, functions, `set -euo pipefail`, argument parsing; the natural next note once a pipeline needs a variable
- [SUGGESTED] `bridge-cli-for-ml-datasets.md` — bridge note: `find`/`awk`/`jq` over JSONL/CSV dataset files, connecting to `ai_ml/` data pipelines

## 🤔 Open Questions
- [ ] Should the notes target **Ubuntu 24.04 specifically** (the real environment) or stay distribution-agnostic? `locate`'s implementation, `ls` colours, and the default `PATH` all differ across distros.
- [ ] How much `rg`/`fd`/`eza`/`bat` should I rely on? They are far better, but absent on a minimal server — is the right policy "standard tools first, modern tools opportunistically"?
- [ ] `tldr` pages are excellent for the first 80% of a command. Should they be a first-class source in this note, or deliberately excluded to avoid shallow learning?
- [ ] What is the actual measured benefit of tab completion on navigation speed, versus typing full paths? Worth benchmarking, or too small to matter?
- [ ] Where does shell scripting (loops, functions, argument parsing) belong — `linux/06-bash-scripting.md`, or should it wait until a real repetitive task appears (as the current backlog says)?
- [ ] Should `dev_foundation/tooling/` exist for `curl`, `jq`, `tmux`, `make`, `strace` — or do they fold into the existing notes?

---
*Navigation is not a preliminary to real work. For a systems engineer, it is the work — and it is the only skill that is required at 3am on an unfamiliar server.*

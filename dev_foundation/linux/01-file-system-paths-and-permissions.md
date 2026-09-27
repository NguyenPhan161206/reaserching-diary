# Linux — File System, Paths & Permissions

**Created Date:** 2026-09-28
**Last Updated:** 2026-09-28
**Tags:** `#Linux`, `#Filesystem`, `#Permissions`, `#DevTools`
**References:** `man 5 path_resolution`, `man 2 chmod`, `man 2 open`
**Status:** Draft
**Difficulty:** Beginner
**Prerequisites:** none
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`

---

## 📌 Key Takeaways (TL;DR)
- There is **one** filesystem tree rooted at `/`. "Drives" (C:, D:) do not exist — but a WSL2 install *adds* Windows drives under `/mnt/c`. That single fact explains most confusion.
- Permissions are **three triples** (user / group / other), each triple being `rwx` → read, write, execute. A *directory* needs `x` to be enterable at all — this is the single most confusing rule for beginners.
- `~` is a shortcut for `$HOME`. `.` is the current directory, `..` is the parent, and **`..` does not stop at your home directory** — it goes up to `/`.

---

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

Before any command: **four mental models** that make the rest predictable.

#### 1.1 One tree, rooted at `/`

Linux has a **single namespace** for storage. `/` is the root of everything: not a folder on another disk, the *top* of the hierarchy. Other disks are **attached** at a directory (mounted), not given their own letter.

```
/                     ← root of everything
├── home/             ← home directories (yours is /home/<user>)
├── etc/              ← system-wide configuration
├── var/              ← variable data: logs, databases, mail
│   └── log/          ← system logs
├── tmp/              ← temporary files, wiped on reboot
├── usr/              ← userland programs (not "user", historically)
├── bin/, sbin/       ← essential binaries
├── opt/              ← third-party installed software
├── mnt/              ← mount points (WSL2: /mnt/c is your Windows C: drive)
└── dev/              ← device files (not real files!)
```

**Why this model matters:** every path you type is a *route* through this tree. When a tool "can't find the file", you are almost always looking in the wrong branch, or the file is mounted somewhere you didn't expect.

> **WSL2 specific:** Windows drives appear at `/mnt/c`, `/mnt/d`, ... That is why `/mnt/c/Users/...` works but `C:\Users\...` does not. Also, WSL2 has its own root filesystem; files written from Linux in `/` are inside the WSL virtual disk, invisible to Windows Explorer. Cross-boundary work belongs in `/mnt/c/...` (slower, I/O-bound) or in `~` (faster, Linux-only).

#### 1.2 Absolute vs relative paths

| Type | Starts with | Meaning | Example |
|------|-------------|---------|---------|
| **Absolute** | `/` | From the root, unambiguous | `/etc/passwd` |
| **Relative** | anything else | From your **current working directory** | `docs/notes.md` |
| **Relative with `~`** | `~` | Your home directory | `~/reaserching-diary` |
| **Current** | `.` | Where you are right now | `.` |
| **Parent** | `..` | One level up | `../..` |

**The trap:** `..` is a *graph operation*, not a boundary. From `/home/me/project`, `../../..` is `/`. From `/`, `..` is still `/`. So a script with `cd ../../..` can silently walk above where you expected.

**The other trap:** a relative path is resolved by the *shell's* current directory, which is remembered **per shell session**. Two terminals can be in different directories at the same time. Always `pwd` before assuming.

#### 1.3 Permissions: three triples, nine bits

Every file and directory carries one 9-bit mode, plus an owner and a group:

```
-rwxr-xr--   1  binhnguyen  binhnguyen  4096  Sep 28 10:00  run.sh
│└┬┘└┬┘└┬┘    │        │              │             │       │
│ │  │  │      │        │              │             │       └─ name
│ │  │  │      │        │              │             └───────── size in bytes
│ │  │  │      │        │              └─────────────────────── group
│ │  │  │      │        └────────────────────────────────────── owner (user)
│ │  │  │      └─────────────────────────────────────────────── link count
│ │  │  └──────────────────────────────────────────────────────── other
│ │  └─────────────────────────────────────────────────────────── group
│ └───────────────────────────────────────────────────────────── user (owner)
└─────────────────────────────────────────────────────────────── file type (- = file, d = directory, l = symlink)
```

| Triple | Who it applies to |
|--------|-------------------|
| `rwx` (1st) | the **owner** (user) |
| `rwx` (2nd) | the **group** |
| `rwx` (3rd) | **others** (everyone else) |

**The rule that makes directories different:** the `x` bit on a **directory means "you may enter/traverse it"**, *not* "you may execute it". Without `x` on a directory, you can still `ls` its name via the parent, but you cannot `cd` into it, and you cannot read its contents — even if it is `rwx`.

This produces the classic confusion: a directory that is `r--r--r--` looks "readable" in `ls -l` but cannot be entered, and every path through it fails with **Permission denied**.

| Bits | On a file | On a directory |
|------|-----------|----------------|
| `r` | read contents | list entry names |
| `w` | modify contents | create / delete / rename entries |
| `x` | execute it (run it) | enter it, and access the entries inside (`stat` it by path) |

**Directory `rwx` shorthand:** `r` + `w` but no `x` = you can see file *names* but not open them. A frequent misconfiguration.

#### 1.4 Symlinks and the difference between "path" and "inode"

- A **path** is a route. A **file** (technically an *inode*: a data record with owner, permissions, timestamps, and block locations) may have **many paths** pointing at it.
- A **symlink** (`l`) is a file whose contents are a path. Reading it makes the kernel follow the path. Delete the target and the symlink dangles.
- A **hard link** is a second directory entry for the *same* inode. The link count (`2` in the example above) is how many names that inode has. The data dies only when the last name is gone.

**Why this matters:** `rm` on a file removes a *name*. If two names exist, your data survives. This is why "I deleted it and it's still there" and "I deleted it and it is gone forever" can both be true, depending on the link count.

### 2. Mechanism (How It Works)

#### 2.1 Path resolution, step by step

When the kernel resolves `/usr/local/bin/tool`, it walks components one at a time, checking **execute permission on every directory it passes through**:

```
1. Start at "/"          → needs x on /
2. "usr"                 → lookup "usr" inside / ; needs x on /
3. "local"               → lookup "local" inside /usr ; needs x on /usr
4. "bin"                 → lookup "bin" inside /usr/local ; needs x on /usr/local
5. "tool"                → final component: needs r on the parent dir to list,
                           and x on the parent to actually open by name
6. If a component is a symlink → kernel resolves the *whole remaining path* against the link target
```

**Consequence:** a single directory without `x` in the middle of the path breaks *everything* below it, no matter how permissive the leaf is. This is also why `chmod -R` from the wrong directory is destructive — it can remove `x` from `/` or `/home`.

#### 2.2 The three states of a file's *name* (and why `chmod` is not a move)

`chmod` (change mode) edits the inode's permission bits. `chown` changes the inode's owner/group. Neither changes the *directory entry* — the name stays where it is. Only `mv` (which, within one filesystem, is a rename of the directory entry) changes the name. Across filesystems, `mv` becomes *copy + delete*, which is why moving a large directory across disks is slow.

**`mv` vs `cp` vs `ln`, mechanically:**

| Command | What the kernel does |
|---------|---------------------|
| `mv a b` (same FS) | rename the directory entry — instant, no data copied |
| `cp a b` | read all data, allocate new inode, write data, copy permissions |
| `ln a b` | create a new directory entry pointing at the *same inode* |
| `ln -s a b` | create a new small file *containing* the path `a` |

#### 2.3 `umask`: the default that surprises you

New files and directories get permissions derived from the request minus the **umask**:

```
new file permission      = 666 − umask     (rw-rw-rw- by default)
new directory permission = 777 − umask     (rwxrwxrwx by default)
```

With the common `umask 022`:
- new file → `644` (`rw-r--r--`)
- new directory → `755` (`rwxr-xr-x`)

**Why this matters:** scripts created by an editor or `git clone` are **not executable** until you `chmod +x`, because the file bit was never set. This is the source of `./script.sh: Permission denied` — and it is a *permissions* problem, not a "shebang problem".

### 3. Command Cheat Sheet

| Command | What it does | When to use | Example / Output |
|---------|--------------|-------------|------------------|
| `pwd` | Print working directory | Before any relative-path command; after `cd` into a new terminal | `/home/binhnguyen/reaserching-diary` |
| `ls -lah` | List, long format, all files, human-readable sizes | Default "what's in here?" — use `-h` always so you see KB/MB not bytes | `drwxr-xr-x 2 user user 4.0K Sep 28 10:00 notes` |
| `ls -ld dir/` | Long format of a directory itself (not its contents) | Check permissions of a directory you cannot enter | `dr-xr-xr-x 2 root root 4.0K . ..` |
| `cd -` | Go to the **previous** directory | Fast toggle between two working dirs | returns to previous path |
| `cd -` (shell builtin, no args) | Go to `$HOME` | Return home from anywhere | `~` |
| `cd /` then `ls -la` | Leave your home before exploring the tree | Prevent accidentally creating junk at `$HOME` | see root listing above |
| `tree -L 2` | Show directory structure 2 levels deep | Get oriented in an unfamiliar project | `.`<br>`├── src`<br>`└── tests` |
| `realpath f` | Resolve to absolute canonical path, following symlinks | Debug what a path *really* points at | `/mnt/c/Users/...` |
| `readlink -f f` | Show symlink target only | Inspect a symlink chain | `/etc/nginx/sites-enabled/default` |
| `mkdir -p a/b/c` | Create nested dirs, no error if they exist | Setting up a project scaffold in one command | creates 3 levels |
| `touch f` | Create empty file / update timestamps | Create a file placeholder; check "when was this touched" | `-rw-r--r-- 1 user user 0 Sep 28 10:05 f` |
| `cp -r src/ dst/` | Recursive copy | Copy a tree; note the trailing `/` on the source | copies `src` *into* `dst` |
| `mv old new` | Rename (or move across filesystems = copy+delete) | Reorganise; instant within one filesystem | — |
| `rm -r dir/` | Recursive delete | Delete a tree — **check `pwd` first** | no output, gone |
| `rm -i f*` | Interactive delete with prompt | When a glob might catch more than intended | `rm: remove file 'a.txt'?` |
| `ln -s target link` | Create a symlink | One config used by many tools; "shortcut" to a dir | `lrwxrwxrwx ... link -> target` |
| `stat f` | Full metadata: size, inode, links, times, numeric mode | "Why can't I read this?" / "are these two files the same inode?" | `Inode: 1320456  Links: 2` |
| `file f` | Detect type by content magic | Unknown binary; check if it is text, archive, ELF | `ELF 64-bit LSB pie executable` |
| `du -sh dir/` | Disk usage of a tree | "Which folder is eating my disk?" | `412M ./node_modules` |
| `df -h` | Free space per **filesystem** | "Why can't I write a 2GB file on a 50GB disk?" (different FS) | `Filesystem Size Used Avail Use%` |
| `chmod u+x f` | Add execute for owner | Make a script runnable | `-rwxr-xr--` |
| `chmod 644 f` | Set exact mode (rw-r--r--) | Normalise a file's permissions | predictable, safe default |
| `chmod 755 d/` | Set directory mode (rwxr-xr-x) | Standard for shared directories | traversable by all, writable by owner |
| `chmod -R u+rwX d/` | Recursive, `X` = add x **only to dirs** | Fix a whole tree without making data files executable | the safe recursive form |
| `chown user:group f` | Change owner and group | After a file arrives from another user / archive | — |
| `chgrp dev f` | Change group only | Share a file with a team group | — |
| `id` | Show my uid, gid, groups | "Which group am I in? why can't I write here?" | `uid=1000(binhnguyen) groups=1000,27(sudo),44(video)` |
| `namei -l /a/b/c` | Show permissions of **every** component of a path | The best debugging tool for Permission denied | finds the exact dir that blocks |
| `getfacl f` | Show full ACL | Fine-grained permissions beyond rwx | `-#effective:r--` reveals masks |
| `umask` | Show current default permissions mask | "Why is this new file `644`?" | `0022` |
| `find . -name '*.md' -type f` | Search by predicate, recursively | Locate a file when you forgot the path | list of matches |
| `find . -type f -size +100M` | Files over 100 MB | Find the bloat before committing it | — |
| `ls -i f` | Show inode number | Confirm two names point to the same data (hard link) | compare inodes |
| `diff <(cmd1) <(cmd2)` | Compare two command outputs | Diff two configs/generated files | — |

### 4. Practical Lab

#### Lab 1 — Read the filesystem like a detective

**Goal:** build the mental model by inspecting, not by memorising.

```bash
# 1. Where am I, and what is up?
pwd
ls -la /
ls -la ..

# 2. What am I as a user? (explains why some dirs are forbidden)
id

# 3. Pick a real path and inspect EVERY component's permissions.
#    This is the "Permission denied" debugging tool.
namei -l /etc/passwd
namei -l /usr/bin/env

# 4. Compare a file and a directory mode side by side.
ls -ld /etc          # directory: dr-xr-xr-x
ls -l  /etc/passwd   # file:      -rw-r--r--

# 5. What can I write to? (the practical permission question)
touch /etc/testfile 2>&1 | head -1
touch ./testfile && echo "home is writable"

# 6. umask explains default modes
umask
touch ./newfile && ls -l ./newfile
```

**Expected output (highlighted lines):**

```
$ pwd
/home/binhnguyen/reaserching-diary

$ id
uid=1000(binhnguyen) gid=1000(binhnguyen) groups=1000(binhnguyen),27(sudo),44(video)

$ namei -l /etc/passwd
 drwxr-xr-x root root /
 drwxr-xr-x root root etc
 -rw-r--r-- root root passwd          ← leaf: readable by all, writable only by root

$ touch /etc/testfile
touch: cannot touch '/etc/testfile': Permission denied     ← NOT because of the file, because of /etc

$ touch ./testfile && echo "home is writable"
home is writable

$ umask
0022
$ ls -l ./newfile
-rw-r--r-- 1 binhnguyen binhnguyen 0 ... newfile     ← 666 − 022 = 644
```

**Verify your understanding:** the `/etc` directory is `drwxr-xr-x` — readable and traversable by all, writable only by `root`. `ls` worked; `touch` did not. Same file path, different permission bit, opposite outcomes.

#### Lab 2 — The `chmod` lifecycle of a script

**Goal:** reproduce and fix `./script.sh: Permission denied`.

```bash
mkdir -p /tmp/perm-lab && cd /tmp/perm-lab

# 1. Create a script the naive way
printf '#!/bin/bash\necho "hello from the script"\n' > hello.sh
ls -l hello.sh                     # -rw-r--r--  ← no execute bit

# 2. Run it the wrong way and the right way
./hello.sh                         # Permission denied (no x bit)
bash hello.sh                      # works: the interpreter reads the file, x not needed

# 3. Add the execute bit — three equivalent ways
chmod +x hello.sh
# chmod u+x hello.sh
# chmod 755 hello.sh
ls -l hello.sh                     # -rwxr-xr--

# 4. Now it runs directly
./hello.sh                         # hello from the script

# 5. Safely fix a whole tree — the capital X matters
mkdir -p proj/bin proj/data
printf '#!/bin/bash\necho ok\n' > proj/bin/run.sh
chmod -R u+rwX proj
ls -l proj/bin/run.sh              # -rwxr-xr-x  ← X gave x to the script
ls -l proj/data                    # drwxr-xr-x  ← X gave x to the directory
ls -l proj/data/README.md          # -rw-r--r--  ← X did NOT make the data file executable

# 6. Clean up
cd ~ && rm -rf /tmp/perm-lab
```

**The lesson in step 5:** `chmod -R +x` (without `u` and `X`) would make *every* file executable, including `README.md` and any `.csv` data. `u+rwX` adds `x` only to directories and to files that already had it somewhere — the safe idiom.

#### Lab 3 — Symlink and hard link, side by side

```bash
mkdir -p /tmp/link-lab && cd /tmp/link-lab
echo "original data" > original.txt

# Hard link: second NAME for the same inode
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt   # note: SAME inode number, link count = 2

# Symlink: a small FILE whose contents are a path
ln -s original.txt symlink.txt
ls -li original.txt symlink.txt   # DIFFERENT inode, type = l

# Delete the original name — hard link survives, symlink dangles
rm original.txt
cat hardlink.txt                   # original data     ← data still exists
cat symlink.txt                    # No such file or directory  ← dangling

cd ~ && rm -rf /tmp/link-lab
```

**The lesson:** `rm` removes a *name*. Hard link = another name for the same data. Symlink = a shortcut that can break.

### 5. Common Pitfalls & Gotchas

| Pitfall | Symptom | Why | Fix |
|---------|---------|-----|-----|
| `rm -rf ./` typed while in the wrong directory | Irrecoverable data loss | `.` is *wherever you are*; `rm -r` has no undo and no trash can | `pwd` first; use `rm -i`; prefer moving to a temp dir over deleting |
| `sudo rm -rf /` style "clean up" | Catastrophic | `sudo` removes the permission safety net that would have stopped you | Never `sudo rm` a path you cannot `ls` first |
| `chmod -R 777` as a fix | "Permission denied" fixed, security destroyed | World-writable means any process on the machine can modify or replace the file | Fix the specific missing bit: `chmod u+x`, `chmod -R u+rwX` |
| `chmod -R` from `/` or `/home` | Broke the whole system (or your shell) | Removing `x` from a parent directory makes everything beneath it unreachable | Test with `namei -l path` before recursive chmod |
| `./script.sh: Permission denied` on a fresh script | Nothing runs | The file bit was never set (umask gives files `644`) | `chmod +x script.sh`, or `bash script.sh` |
| Shebang points to a missing interpreter | `bad interpreter: No such file` | `#!/usr/bin/python` when Python is at `/usr/bin/python3` | `head -1 script.sh` to check |
| `cp -r src dst` then "where is src?" | `dst/src` instead of `dst` | `cp -r src/ dst/` (trailing slash) copies *contents*, not the folder | Remember the trailing-slash rule; verify with `ls` |
| Editing a file as `root`, then not being able to edit it again | "Permission denied" on your own file | You created it via `sudo`, so the owner is `root` | `sudo chown -R $USER:$USER path` |
| `cat` a 2GB log | Terminal freezes, OOM | `cat` streams the whole file into your terminal buffer | `less file`, or `head`/`tail`, or `grep` for what you need |
| `mv` a big directory across filesystems and it hangs | Very slow operation | Cross-filesystem `mv` = full copy + delete | Same-filesystem moves are instant; check with `df` |
| Windows line endings (`CRLF`) in a Linux script | `$'\r': command not found` | The `\r` becomes part of the command name | `dos2unix file` or `sed -i 's/\r$//' file` |
| Relative path in a script that worked when run by hand | `No such file or directory` | Scripts start with CWD = caller's dir, not the script's dir | Use `$(dirname "$0")` or absolute paths |
| `rm file` then wondering where it went | Gone | No trash can on Linux; `rm` unlinks immediately | `gio trash file` (desktop) or move instead of delete |

### 6. Comparison Table

| Aspect | Linux / Unix | Windows |
|--------|--------------|---------|
| Root | Single tree at `/` | Drive letters per volume (`C:\`, `D:\`) |
| Path separator | `/` | `\` (forward slash mostly works) |
| Case sensitivity | **Case-sensitive** (`File.txt` ≠ `file.txt`) | Case-insensitive by default |
| Permissions | POSIX bits (user/group/other, rwx) | NTFS ACLs + inheritable permissions |
| Deleting | Immediate, no trash can | Recycle Bin by default |
| Executable file | Set the execute bit | File extension (`.exe`) |
| Symlink | First-class; cheap and common | Requires Developer Mode or admin |
| Line endings | `\n` (LF) | `\r\n` (CRLF) |
| Path env var | `$PATH` | `%PATH%` (per-process) |
| Shell defaults | Bash/Zsh/Fish | PowerShell (also legacy cmd.exe) |

---

## 🔗 Tech Cross-References
- **Git** — a repository *is* a directory of files: `.git/objects`, `refs`, and `HEAD` are plain files obeying these permissions. `git config core.fileMode` exists precisely because of this execute-bit behaviour. Windows↔Linux line-ending problems (`core.autocrlf`, `.gitattributes`) are a filesystem-convention clash.
- **Docker** — a container uses Linux **namespaces** to give a process its own view of this filesystem, plus a **union filesystem** (overlay layers) where the image is read-only and your writes live in a top layer. Volumes are *mounts* — literally the mechanism in §1.1. → [`../docker/`](../docker/README.md)
- **YAML** — `docker-compose.yml` and `.github/workflows/*.yml` are configuration *files*; you inspect them with `cat`/`less` and they obey `644`. See [`../yaml_config/`](../yaml_config/README.md).
- **Shell vs PowerShell** — same goal (automate the OS), opposite design: Unix composes small text-processing programs with pipes, Windows passes rich .NET objects between cmdlets. Understanding `chmod`/`chown` is meaningless in PowerShell — it has `icacls`/`Get-Acl` instead.
- **DSA** — a solution script you `chmod +x` once, then run in a loop. `time ./a.out` and `| less` are how complexity claims get *measured* instead of assumed. → [`../dsa/01-learning-roadmap.md`](../dsa/01-learning-roadmap.md)
- **Databases** — `psql`/`mysql` are ordinary programs installed in `/usr/bin`; a database server is a service writing into `/var/lib`. The `postgres` Docker image chowns `/var/lib/postgresql` at build time for exactly this reason. → [`../databases/`](../databases/README.md)
- **Security** — permissions and the principle of least privilege: the reason `sudo` is separate from your user account, and why a production web server runs as `www-data`, not `root`.

## 🧪 Practice Tasks
> No solutions provided on purpose. Do them first, then ask AI to grade — or compare with a fresh note afterwards.

- [ ] **Task 1 — Permission autopsy.** Reproduce a `Permission denied` in three different ways (missing `x` on a directory, wrong group, root-owned file), then diagnose each with `namei -l` and `id`. Write one sentence per case explaining which bit was missing and *where*.
- [ ] **Task 2 — Mode normalizer.** Write a shell script `normalize.sh` that, given a directory, sets directories to `755` and regular files to `644`, and adds `u+x` only to `*.sh` files. Test it on a tree containing a subdirectory, a data file, and two scripts. Verify with `ls -lR`.
- [ ] **Task 3 — umask experiment.** Record `umask`; create a file and a directory; record their modes. Then set `umask 077` and repeat. Explain the numbers arithmetically (which bits were masked off, and which of the nine bits each result has).
- [ ] **Task 4 — Link semantics.** Create a file, a hard link, and a symlink to it. Delete (a) only the original name, (b) the hard link, (c) the symlink — one case per reset. For each, state which data still exists and which inode numbers were involved.
- [ ] **Task 5 — Find the space.** In a directory of your choice, use `du -sh` per subdirectory to find the largest, then `find -size +50M` to list the big files inside it. Report the top three with sizes, and explain why `df -h` might show a *different* number than the sum of `du`.
- [ ] **Task 6 — Dockerfile ownership check (Docker bridge).** Read a small public `Dockerfile` (e.g. the official `postgres` image). Identify every `chown`/`chmod`/`USER` line and write, for each, which Linux permission problem it is solving. This is graded as *both* a Docker and a Linux exercise.

---

## 📝 Research Journey
- **Why:** I kept hitting errors I could not explain — `./script.sh: Permission denied`, "works on my machine", a file I could not edit, `mv` taking forever. All of them are this note. I need the filesystem to stop being magic before Docker (which is namespaces over this filesystem) or Git (which is files in this filesystem) make any sense.
- **Struggle:** The `x` bit meaning different things for files vs directories. Reading `dr-xr-xr-x` and thinking "readable, so I can look inside" — and then failing. It only clicked when `namei -l` showed *which* directory in a path was blocking me. Also, `umask` looked arbitrary until I saw `666 − 022 = 644`.
- **Aha moment:** A path is not a string, it is a **sequence of permission checks**. The moment I stopped asking "does this file exist?" and started asking "does every directory on the way to it let me pass?" (`namei -l` became my first debugging command. Permission errors stopped being mysterious.
- **Career link:** For an AI Engineer this is the layer under everything: a model server's files, a dataset's permissions, a container's volumes, a CI artifact. When a training job fails at 3 AM with "Permission denied", the ability to read `namei -l` and `ls -l` output correctly is worth more than knowing another framework's API.

## 🔗 Related Topics
- [[dev_foundation/linux/README.md]] — folder guide and roadmap
- [[dev_foundation/linux/03-text-processing-and-pipes.md]] — the commands that make the cheat sheet above actually useful in a pipeline
- [[git_github/01-version-control-concepts.md]] — Git's three areas are directory entries on this filesystem
- [[git_github/03-core-daily-workflow.md]] — daily commands that assume this note
- [[dev_foundation/docker/README.md]] — namespaces and overlay filesystems build on §1.1
- [[dev_foundation/dsa/01-learning-roadmap.md]] — where `chmod +x` + `time ./a.out` get used daily
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there
- [SUGGESTED] `02-navigation-and-core-cli.md` — `cd`/`find`/`locate`/`man` efficiency; the natural next note
- [SUGGESTED] `bridge-wsl2-filesystem-boundary.md` — bridge note: Windows↔Linux path translation and I/O cost across `/mnt/c` (directly affects this machine's setup)

## 🤔 Open Questions
- [ ] Should the WSL2 filesystem boundary (Linux vs `/mnt/c` performance) be its own note, or a section in `02-navigation-and-core-cli.md`?
- [ ] How do ACLs (`getfacl`/`setfacl`) change the "three triples" model — and are they common enough to matter?
- [ ] Where do extended attributes / `chattr +i` (immutable files) fit — useful for config protection, overkill for now?
- [ ] What is the practical difference between `rm` and `shred` for a file with secrets in it?

---
*Linux has no trash can, no undo, and no wizard. That is why it is easy to learn and easy to destroy — the filesystem shows you the truth, immediately.*

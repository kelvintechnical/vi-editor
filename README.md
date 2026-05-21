# Lab 26: Command Mode and Insert Mode in `vi` / `vim`

**Series:** File Operations & Shell Fundamentals · **Lab 26 of the Novice → RHCA path**  
**Certifications covered:** RHCSA EX200 (every editing task), RHCE EX294 (playbook authoring on the control node), CKA (manifest editing, `kubectl edit`), RHCA building blocks (RH342 fast recovery edits, RH358 service configs, RH236 brick config edits)  
**Prerequisite:** Labs 05–25 (filesystem, `cat`, `less`, `grep`, `sed`, `awk`)  
**Time Estimate:** 60–80 minutes  
**Difficulty arc:** Tasks 1–6 foundation · 7–13 practical · 14–18 advanced · 19–20 exam-realistic

---

## 🎯 Objective

By the end of this lab you will edit files in `vi` (or `vim`) **without thinking about which key you're pressing**. You'll switch between Command, Insert, and Last-line modes fluidly, you'll move, search, replace, save, and quit at speed — and you'll never again `:q!` away a real change by accident. `vi` is the only editor guaranteed to be on every Linux system, every rescue shell, every container. Mastering it is non-negotiable.

---

## 🧠 Concept: `vi` Is a State Machine

Most editors have one mode: type letter → letter appears. `vi` has **three** primary modes. Every keystroke is interpreted according to the mode you're in. This is what makes `vi` confusing for 30 minutes and incredible for 30 years.

```
                    ╔═══════════════════════════╗
                    ║      COMMAND MODE         ║  ← starting mode
                    ║  (movement, deletion,     ║
                    ║   yank, paste, search)    ║
                    ╚═══════════════════════════╝
              i / a / o / I / A / O      ⇡ Esc
                          ▼               ⇡
                    ╔═══════════════════════════╗
                    ║       INSERT MODE         ║
                    ║   (typing inserts text)   ║
                    ╚═══════════════════════════╝

                      :                  ⇡ Esc / Enter
                      ▼                  ⇡
                    ╔═══════════════════════════╗
                    ║      LAST-LINE MODE       ║  (ex commands)
                    ║   (:w, :q, :s/A/B/g, :!)  ║
                    ╚═══════════════════════════╝
```

### Rule of thumb

| Need to… | Mode |
|---|---|
| Move, delete, copy, paste | Command |
| Type text | Insert |
| Save, quit, search-replace globally | Last-line |
| Anywhere, any time → escape to Command | **Press `Esc`** |

> **Senior-engineer reflex:** When in doubt, press `Esc`. You're now in Command mode. From there, everything is possible.

### `vi` vs. `vim` vs. `nvim`

| Editor | What it is | When you'll meet it |
|---|---|---|
| `vi` | The original. POSIX-mandated. | Rescue shells, busybox, very old systems |
| `vim` | "Vi IMproved" — modern superset (syntax, undo branches, plugins) | Default `vi` on RHEL, Rocky, Ubuntu |
| `nvim` | Neovim — Lua-scriptable fork | Increasingly common, fully `vi`-compatible |

On RHEL: `which vi` typically reports `/usr/bin/vi` symlinking to `/usr/libexec/vi`, and `vim` is `/usr/bin/vim`. The minimal `vi` is intentionally small for rescue. Everything in this lab works in both unless noted.

---

## 📚 Key & Command Reference

### Entering Insert mode (from Command)

| Key | What it does |
|---|---|
| `i` | Insert **before** the cursor |
| `I` | Insert at the **start** of the current line |
| `a` | Append **after** the cursor |
| `A` | Append at the **end** of the line |
| `o` | Open a new line **below** and enter insert |
| `O` | Open a new line **above** and enter insert |
| `s` | **S**ubstitute the char under cursor (delete + insert) |
| `S` | Substitute the whole line |
| `R` | Replace mode (overtype) |
| `Esc` | Leave Insert mode → back to Command |

### Movement (Command mode)

| Key | Motion |
|---|---|
| `h` `j` `k` `l` | Left, Down, Up, Right |
| `0` | Start of line |
| `^` | First non-blank of line |
| `$` | End of line |
| `w` / `W` | Next word / WORD (space-separated) |
| `b` / `B` | Previous word / WORD |
| `e` / `E` | End of next word / WORD |
| `gg` | First line of file |
| `G` | Last line of file |
| `NG` or `:N` | Go to line N |
| `Ctrl-d` / `Ctrl-u` | Half-page down / up |
| `Ctrl-f` / `Ctrl-b` | Full-page down / up |
| `%` | Jump to matching bracket `( ) { } [ ]` |
| `''` (two apostrophes) | Jump back to previous position |

### Editing (Command mode)

| Key | Action |
|---|---|
| `x` | Delete char under cursor |
| `X` | Delete char before cursor |
| `dd` | Delete (cut) line |
| `Ndd` | Delete N lines |
| `dw` | Delete word |
| `d$` or `D` | Delete to end of line |
| `d0` | Delete to start of line |
| `yy` | **Y**ank (copy) line |
| `Nyy` | Yank N lines |
| `yw` | Yank word |
| `p` | Paste **after** cursor (line below for line-yank) |
| `P` | Paste **before** cursor |
| `u` | Undo |
| `Ctrl-r` | Redo |
| `.` | Repeat the last edit |
| `r<char>` | Replace one character |
| `~` | Toggle case |
| `J` | Join current line with the next |

### Searching (Command mode)

| Key | Action |
|---|---|
| `/PATTERN` | Forward search |
| `?PATTERN` | Backward search |
| `n` / `N` | Next / previous match |
| `*` / `#` | Search for word under cursor forward / backward |
| `:noh` | Clear current search highlight |

### Last-line (ex) commands

| Command | Action |
|---|---|
| `:w` | Write (save) |
| `:w FILE` | Save as FILE |
| `:q` | Quit |
| `:wq` or `ZZ` or `:x` | Save and quit |
| `:q!` | Quit without saving |
| `:wa` / `:qa` / `:wqa` | All buffers |
| `:e FILE` | Open another file |
| `:set nu` / `:set nonu` | Show / hide line numbers |
| `:set paste` / `:set nopaste` | Disable / enable auto-indent for pasting |
| `:s/A/B/` | Substitute first match on current line |
| `:s/A/B/g` | Substitute all matches on current line |
| `:%s/A/B/g` | Substitute everywhere |
| `:%s/A/B/gc` | Substitute everywhere with **c**onfirmation |
| `:N,Ms/A/B/g` | Substitute in lines N–M |
| `:N` | Jump to line N |
| `:!CMD` | Run shell command |
| `:r FILE` | Read FILE into the buffer |
| `:r !CMD` | Read the output of CMD into the buffer |

### Visual mode (Vim only — bonus)

| Key | Action |
|---|---|
| `v` | Character-wise selection |
| `V` | Line-wise selection |
| `Ctrl-v` | **Block** (column) selection |
| `y` / `d` / `>` / `<` | Yank / delete / indent / dedent the selection |

---

## 🛣️ RHCA Pathway Sidebar

| Cert level | Why this lab matters |
|---|---|
| **Foundation** | The only editor present **everywhere** — rescue shells included |
| **RHCSA EX200** | Every config-edit task uses `vi` (some allow `nano`, but `vi` is universal) |
| **RHCE EX294** | Playbook YAML authoring on the control node |
| **CKA** | `kubectl edit deploy/X` launches `vi` by default — must be fluent |
| **RHCA — RH342 (Troubleshooting)** | `:set paste` and `:r !CMD` are forensic gold |
| **RHCA — RH358 (Services)** | Editing service unit overrides — `systemctl edit SERVICE` opens `vi` |
| **RHCA — RH236 (Storage)** | Brick option files edited node-by-node in `vi` |

---

## 🔧 The 20 Tasks

> Each task ends with three short callouts: **Keys / commands**, **Output decoded**, and **Troubleshoot**.

---

### Task 1 — Open a file and recognize the modes

**Purpose:** Open a test file. Confirm you start in Command mode.

```bash
mkdir -p ~/vi-lab && cd ~/vi-lab
cat > demo.txt <<'EOF'
host = web01
port = 80
ssl  = off
log  = /var/log/app.log
EOF
vi demo.txt
```

Inside `vi`:

```
(do NOT press any letter yet)
:        (you should see a `:` appear at the bottom — last-line mode)
Esc      (back to Command)
i        (notice "-- INSERT --" at the bottom)
Esc      (back to Command)
:q       (quit without saving — you didn't change anything)
```

**Keys / commands**

| Key | Action |
|---|---|
| `vi FILE` | Open file (starts in Command mode) |
| `:` | Enter last-line mode |
| `i` | Enter Insert mode |
| `Esc` | Back to Command |
| `:q` | Quit (when no unsaved changes) |

**Output decoded**

| Element | Meaning |
|---|---|
| Top-left line content | Cursor is on the first line |
| Bottom-left `:` | Last-line mode is active |
| Bottom-left `-- INSERT --` | Insert mode is active |
| No status text | Command mode (the default) |

**Why a sysadmin needs this:** Mode awareness is everything. If your keystrokes "don't work," you're in the wrong mode.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Letters appearing on first key press | The mini `vi` on RHEL minimal images sometimes starts in Insert — press `Esc` first |
| `Esc` doesn't seem to do anything | On some terminals it lags — try `Ctrl-[` (same as Esc) |

---

### Task 2 — Insert text with `i`, `a`, `o`

**Purpose:** Practice the three primary entries to Insert mode.

```bash
vi ~/vi-lab/demo.txt
```

Inside `vi`:

```
i                           (enter Insert)
[type "BEGIN " then Esc]    (text added before the first 'h' on line 1)
A                           (append at END of current line)
[type " # primary" then Esc]
o                           (open new line BELOW)
[type "tls = on" then Esc]
:wq
```

`cat ~/vi-lab/demo.txt` afterward:

```
BEGIN host = web01 # primary
tls = on
port = 80
ssl  = off
log  = /var/log/app.log
```

**Keys**

| Key | Action |
|---|---|
| `i` | Insert before cursor |
| `A` | Append at end of line |
| `o` | Open new line below |
| `Esc` | Return to Command |
| `:wq` | Save and quit |

**Output decoded**

| Line | What happened |
|---|---|
| `BEGIN host = web01 # primary` | `i` prepended, `A` appended |
| `tls = on` | `o` opened a new line below the first |

**Why a sysadmin needs this:** Faster than navigating to a column with arrows.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Typed `a` mid-word and the char landed wrong | `a` appends AFTER the cursor — use `i` for BEFORE |
| Forgot `Esc` and typed `:wq` literally | Press `u` (undo), then `Esc` properly |

---

### Task 3 — Navigate without arrow keys: `h j k l`

**Purpose:** `hjkl` move the cursor without leaving the home row.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
gg          (go to top)
j j j       (down 3 lines)
$           (end of line)
0           (start of line)
^           (first non-blank)
w w         (forward two words)
b           (back one word)
G           (last line)
:1          (jump to line 1)
:q
```

**Keys**

| Key | Action |
|---|---|
| `h` | Left |
| `j` | Down (mnemonic: jelly falls) |
| `k` | Up |
| `l` | Right |
| `0` / `^` / `$` | Line start / first non-blank / end |
| `w` / `b` | Word forward / back |
| `gg` / `G` / `:N` | Top / bottom / jump to line N |

**Output decoded**

| Element | Meaning |
|---|---|
| Cursor moves precisely | Hand stays on home row |
| Status bar | Shows current line/column (in vim, with `:set ruler`) |

**Why a sysadmin needs this on RHCSA EX200:** Some exam terminals lack functioning arrow keys (especially over slow SSH). `hjkl` always works.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Letters typed instead of moving | You were in Insert — `Esc` first |
| Cursor stays put | Confirm you're in Command mode |

---

### Task 4 — Delete characters and lines

**Purpose:** Three deletes you'll use constantly: `x`, `dd`, `dw`.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
gg                  (top)
x x x               (delete first 3 chars — "BEG" goes)
dd                  (delete current line)
dw                  (delete next word)
2dd                 (delete next 2 lines)
u                   (undo)
u                   (undo again — multi-level undo in vim)
:q!                 (quit without saving — your file is safe)
```

**Keys**

| Key | Action |
|---|---|
| `x` | Delete char under cursor |
| `dd` | Delete the line |
| `Ndd` | Delete N lines |
| `dw` | Delete word |
| `D` or `d$` | Delete to end of line |
| `u` | Undo |
| `:q!` | Quit without saving |

**Output decoded**

| Element | Meaning |
|---|---|
| File buffer after edits | Test changes that you can undo |
| `:q!` | Discard ALL changes |

**Why a sysadmin needs this:** Quick line cleanup. Combined with `.` (repeat) you can rip through cleanup tasks.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Stuck after `:q` with `No write since last change` | Either `:wq` (save) or `:q!` (discard) |

---

### Task 5 — Copy and paste with `yy` and `p`

**Purpose:** Yank a line, paste it elsewhere. This is the `vi` equivalent of `Ctrl-C` / `Ctrl-V`.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
gg              (top)
yy              (yank current line)
G               (go to last line)
p               (paste — appears BELOW)
3yy             (yank next 3 lines)
gg              (top)
P               (paste BEFORE — 3 lines inserted above line 1)
:wq
```

**Keys**

| Key | Action |
|---|---|
| `yy` | Yank (copy) the line |
| `Nyy` | Yank N lines |
| `yw` | Yank word |
| `p` | Paste after |
| `P` | Paste before |
| `"ayy` / `"ap` | Use register `a` to keep multiple clipboards |

**Output decoded**

| Element | Meaning |
|---|---|
| Line duplicated at end | Paste below |
| 3 lines duplicated at top | Multi-line yank + paste before |

**Why a sysadmin needs this on RHCSA EX200:** Duplicating a configuration block, then editing only the few lines that differ.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Paste lands inside a line | Yanked with `yw` (word) → pastes at cursor; for line-paste use `yy` |
| Mouse paste mangled formatting | Use `:set paste` first (Task 11) |

---

### Task 6 — Save and quit, with grace

**Purpose:** Five command combinations cover 100% of save/quit needs.

```bash
vi ~/vi-lab/demo.txt
```

Try each variant (open, modify a char with `x`, then test the variant):

| Command | Effect |
|---|---|
| `:w` | Save, keep editing |
| `:w newname.txt` | Save as a new file |
| `:wq` | Save and quit |
| `:x` | Save and quit (only writes if changes) |
| `ZZ` | Same as `:x` — no colon needed |
| `:q` | Quit (errors out if unsaved changes) |
| `:q!` | Quit and **discard** changes |
| `:wqa` | Save and quit ALL buffers |

**Output decoded**

| Element | Meaning |
|---|---|
| Bottom-line "demo.txt 4L, 64C written" | Successful save |
| "No write since last change" | You tried `:q` with unsaved edits |

**Why a sysadmin needs this:** `:x` and `ZZ` are the muscle-memory choices. `:q!` saves you in an emergency.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `E45: 'readonly' option is set` | Use `:w!` (force write) — but check why the file was opened readonly (was the path actually `sudoedit`-required?) |
| `:wq` gives `Permission denied` | Need `sudo`. Exit with `:q!`, reopen with `sudo vi`, or write through `tee`: `:w !sudo tee %` |

---

### Task 7 — Search forward and backward

**Purpose:** `/` is forward, `?` is backward — exactly like `less` (Lab 20).

```bash
sudo vi /etc/ssh/sshd_config
```

Inside:

```
/PermitRoot         (Enter — jumps to first match)
n                   (next match)
N                   (previous match)
?Port               (search backward)
n
:noh                (clear highlight)
:q
```

**Keys**

| Key | Action |
|---|---|
| `/PATTERN` | Forward search (regex) |
| `?PATTERN` | Backward search |
| `n` / `N` | Next / previous match |
| `*` / `#` | Word-under-cursor forward / backward |
| `:noh` | Clear highlight |

**Output decoded**

| Element | Meaning |
|---|---|
| Cursor jumps | To first match |
| Subsequent matches highlighted | Multiple hits |
| `:noh` | Stops the highlight (next `/` reactivates it) |

**Why a sysadmin needs this on RHCSA EX200:** Find the setting you need to change without scrolling.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Search case-sensitive when you wanted insensitive | `:set ic` (ignorecase) |
| Highlight stays | `:noh` or `:set nohlsearch` |

---

### Task 8 — Replace text on the current line: `:s/A/B/`

**Purpose:** Substitute exactly like `sed` — but interactively, scoped to where you are.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
gg
/port
:s/80/8080/
n
:s/web01/web02/
:wq
```

**Commands**

| Command | Action |
|---|---|
| `:s/A/B/` | First match on current line |
| `:s/A/B/g` | All matches on current line |
| `:s/A/B/gi` | Case-insensitive |

**Output decoded**

| Element | Meaning |
|---|---|
| First `:s` | `80` → `8080` on the line under cursor |
| Second `:s` | `web01` → `web02` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `E486: Pattern not found` | The pattern isn't on the current line — navigate first or use `:%s` (Task 9) |

---

### Task 9 — Whole-file replace: `:%s/A/B/g` with confirmation

**Purpose:** Global substitution. Adding `c` asks you per match — invaluable for tricky edits.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
:%s/web01/web42/gc
y         (yes to first match)
n         (no to next)
a         (yes to all remaining)
:wq
```

**Commands**

| Command | Action |
|---|---|
| `:%s/A/B/g` | Substitute ALL `A` with `B` in the whole file |
| `:%s/A/B/gc` | Same but **c**onfirm each |
| `:N,Ms/A/B/g` | Substitute only in lines N–M |
| `y` / `n` / `a` / `q` / `l` | At each confirmation: yes / no / all / quit / last |

**Output decoded**

| Element | Meaning |
|---|---|
| Match highlighted | Awaiting your `y`/`n`/`a` |
| Bottom-line summary | "N substitutions on M lines" |

**Why a sysadmin needs this on RHCSA EX200:** Standard rename operation — far safer with `c` confirmation.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Forgot `g` | Only first match per line replaced |
| Slash in pattern/replacement | Use different delim: `:%s#/old/path#/new/path#g` |

---

### Task 10 — Undo, redo, and the repeat `.` command

**Purpose:** `u` undoes; `Ctrl-r` redoes; `.` repeats the last change. The dot is `vi`'s secret weapon.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
gg
dd          (delete first line)
.           (repeat — delete the new first line)
.           (repeat again)
u           (undo)
u
u
Ctrl-r      (redo)
:wq
```

**Keys**

| Key | Action |
|---|---|
| `u` | Undo (vim supports many levels) |
| `Ctrl-r` | Redo |
| `U` | Undo all changes on current line (POSIX vi) |
| `.` | Repeat the **last edit** |

**Output decoded**

| Element | Meaning |
|---|---|
| Three lines deleted via two dots | `.` repeats whichever edit you did last |
| Restored via `u`s | Full history available |
| Final state | After `Ctrl-r` |

**Why a sysadmin needs this:** Repeated patterns (delete every 3rd line, etc.) become two keystrokes — `dd` then `.`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `.` doesn't repeat what you expected | `.` only repeats "edits" (text changes), not motions |

---

### Task 11 — Paste from outside without auto-indent madness: `:set paste`

**Purpose:** Pasting code with `Ctrl-Shift-V` into a `vi` session with `autoindent` produces staircase indentation. `:set paste` turns indent off temporarily.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
:set paste
i
[paste a multi-line block from your terminal clipboard]
Esc
:set nopaste
:wq
```

**Commands**

| Command | Action |
|---|---|
| `:set paste` | Disable auto-indent, smart-indent, abbreviations |
| `:set nopaste` | Restore them |
| `:set pastetoggle=<F2>` | Bind a toggle key (vim) |

**Output decoded**

| Element | Meaning |
|---|---|
| Pasted content keeps original indentation | No more staircase |

**Why a sysadmin needs this on RHCE EX294:** Pasting YAML into a playbook — indentation matters; `paste` mode saves the day.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Paste still wrong | Some terminals translate tabs; consider `xclip`/`xsel` or terminal "bracketed paste" |
| Forgot to `:set nopaste` after | Subsequent typing also won't autoindent — flip it back |

---

### Task 12 — Visit another file, run a shell command, read in output

**Purpose:** Three `ex` commands that turn `vi` into a small environment: `:e`, `:!`, `:r !`.

```bash
vi ~/vi-lab/demo.txt
```

Inside:

```
:e ~/vi-lab/notes.txt     (creates if missing; switches buffer)
i Hello notes Esc
:wq
vi ~/vi-lab/demo.txt
:!date                      (run shell `date`, paged output)
:r !date                    (insert `date` output into buffer)
:r ~/vi-lab/notes.txt       (read the file into the buffer)
:wq
```

**Commands**

| Command | Action |
|---|---|
| `:e FILE` | Edit another file |
| `:bn` / `:bp` | Next / previous buffer |
| `:!CMD` | Run shell command — show output, then return |
| `:r !CMD` | Insert output of CMD into buffer at cursor |
| `:r FILE` | Insert FILE's contents at cursor |

**Output decoded**

| Element | Meaning |
|---|---|
| `:!date` | Output briefly displayed; press Enter to return |
| `:r !date` | A new line with today's date inserted |
| `:r FILE` | File appended at cursor location |

**Why a sysadmin needs this on RHCA RH342:** Stamp logs with `:r !date` while editing a runbook.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `:!CMD` returns to garbled screen | Press `Ctrl-l` to redraw |
| Shell command refuses TTY | Some interactive commands (e.g., `less`) need a terminal — won't work via `:!` |

---

### Task 13 — Jump to a specific line: `:N` and `NG`

**Purpose:** Pair with `grep -n` (Lab 22) to jump directly to a problem line.

```bash
grep -n PermitRootLogin /etc/ssh/sshd_config
sudo vi /etc/ssh/sshd_config
```

Inside `vi`:

```
:40        (jump to line 40)
40G        (alternative — Command mode)
:set nu    (show line numbers)
gg
:set nonu
:q
```

**Commands**

| Token | Meaning |
|---|---|
| `:N` | Jump to line N |
| `NG` | Same, in Command mode |
| `gg` | Line 1 |
| `G` | Last line |
| `:set nu` | Show line numbers |
| `:set nonu` | Hide line numbers |
| `:set rnu` | Relative line numbers (vim) |

**Output decoded**

| Element | Meaning |
|---|---|
| Cursor lands on line 40 | Precision jump |
| Line numbers shown | Left margin |

**Why a sysadmin needs this on RHCSA EX200:** Combined with `grep -n`, this is the fastest config-edit pattern: `grep -n KEY FILE; vi +N FILE`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `vi +N FILE` opens at top | Older `vi` may need `vi -c ":N" FILE` |

---

### Task 14 — Open at a pattern: `vi +/PATTERN file`

**Purpose:** Skip the open-then-search step.

```bash
sudo vi +/PermitRootLogin /etc/ssh/sshd_config
```

Cursor lands on the first match.

**Switches**

| Token | Meaning |
|---|---|
| `+N` | Open at line N |
| `+/PATTERN` | Open at first match of PATTERN |
| `+CMD` | Run any ex command on open — e.g., `+'set nu'` |

**Output decoded**

| Element | Meaning |
|---|---|
| Cursor on first match | Saves keystrokes |

**Why a sysadmin needs this:** Scripts that drop you into a known editing location: `vi +/server_name nginx.conf`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Pattern not present | Opens at top — that's expected |

---

### Task 15 — Indent/dedent blocks with `>>`, `<<`, and visual mode

**Purpose:** Reformat code/YAML quickly. Critical for RHCE EX294.

```bash
cat > ~/vi-lab/play.yml <<'EOF'
- hosts: all
  tasks:
  - name: install
    package:
      name: nginx
EOF
vi ~/vi-lab/play.yml
```

Inside:

```
gg
V              (line-visual select; cursor on line 1)
3j             (extend selection 3 lines down — total 4 lines selected)
>              (indent the selection one shiftwidth)
gg
V3j
<              (dedent)
:wq
```

**Keys**

| Key | Action |
|---|---|
| `>>` | Indent current line |
| `<<` | Dedent current line |
| `N>>` | Indent N lines |
| `V` (vim) | Line-wise visual mode |
| `>` / `<` after visual | Indent / dedent selection |
| `=` after visual | Auto-indent selection (needs filetype detection) |
| `:set shiftwidth=2` / `:set tabstop=2` / `:set expandtab` | Two-space soft tabs |

**Output decoded**

| Element | Meaning |
|---|---|
| Lines shift right/left by `shiftwidth` | YAML structure restored |

**Why a sysadmin needs this on RHCE EX294:** YAML and Ansible roles live or die by correct indentation.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Tabs sneak in | `:set expandtab` ensures spaces |
| Wrong width | Adjust `:set shiftwidth=N` |

---

### Task 16 — Recover from a crashed session: swap files

**Purpose:** `vim` creates a `.swp` file. If your session crashes (SSH drop), reopening offers recovery.

```bash
vi ~/vi-lab/demo.txt
i hello-recovery Esc
# Simulate crash: kill the shell from another terminal:
#   pkill -9 -f 'vi.*demo.txt'
# Reopen:
vi ~/vi-lab/demo.txt
```

You'll see a prompt offering: `(O)pen Read-Only, (E)dit anyway, (R)ecover, (D)elete it, (Q)uit, (A)bort`.

**Keys / commands**

| Choice | Action |
|---|---|
| `R` | Recover the lost edits from the swap file |
| `D` | Delete the swap (only if you're sure the original is preserved) |
| `O` | Open read-only to inspect |
| `Q` | Quit |

**Output decoded**

| Element | Meaning |
|---|---|
| Swap-file warning | Another vi had this file open or crashed |

**Why a sysadmin needs this on RHCA RH342:** SSH sessions die. Recovery prevents lost work.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Bogus swap warning | Confirm no other process: `ls -la ~/.demo.txt.swp` then `rm` it if safe |
| Want swap elsewhere | `:set directory=~/.vim/swap` |

---

### Task 17 — Edit a file you opened without `sudo`

**Purpose:** You opened `/etc/ssh/sshd_config` as a regular user and made edits. `:w` fails. Save without losing changes.

```bash
vi /etc/ssh/sshd_config
# Make a small edit, then:
:w !sudo tee % > /dev/null
:q!
```

**Commands**

| Token | Meaning |
|---|---|
| `:w !CMD` | Pipe the buffer to CMD (your edits are stdin) |
| `tee %` | `%` is the current filename; `tee` writes through sudo |
| `> /dev/null` | Suppress the echoed contents |
| `:q!` | Quit (buffer is now stale relative to disk) |

**Output decoded**

| Element | Meaning |
|---|---|
| sudo prompt | Authenticate |
| `:q!` | Discard the now-out-of-date buffer; the file on disk has your edits |

**Why a sysadmin needs this on RHCSA EX200:** Saves you the "exit, sudo vi, redo edits" cycle. Use sparingly — `sudoedit` is the recommended workflow.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| sudo asks for password but you can't type | Some terminals consume Esc — press `Ctrl-c` then redo `:w !sudo …` |
| sudoedit alternative | Run `sudoedit /etc/ssh/sshd_config` from the shell — opens in `$EDITOR` and saves atomically |

---

### Task 18 — Persist your preferences with `~/.vimrc`

**Purpose:** A small `~/.vimrc` saves time on every session. Keep it minimal and exam-safe.

```bash
cat > ~/.vimrc <<'EOF'
set nocompatible        " full vim features
set number              " line numbers
set ruler               " cursor position
set showcmd             " show partial command in bottom right
set incsearch           " incremental search
set hlsearch            " highlight all matches
set ignorecase smartcase
set tabstop=2 shiftwidth=2 expandtab
set autoindent
set wildmenu            " ex-command tab completion
syntax on
filetype plugin indent on
" Quick toggle paste mode
set pastetoggle=<F2>
EOF
vi ~/vi-lab/demo.txt
```

Open any file — line numbers and syntax highlighting appear immediately.

**Commands**

| Line | Meaning |
|---|---|
| `set nocompatible` | Use vim features, not strict POSIX vi |
| `set number` | Show line numbers |
| `set ignorecase smartcase` | Search insensitive unless pattern has uppercase |
| `set tabstop=2 …` | 2-space soft tabs (YAML-friendly) |
| `syntax on` | Color syntax |
| `pastetoggle=<F2>` | Press F2 to flip paste mode |

**Output decoded**

| Element | Meaning |
|---|---|
| Numbers in margin | `set number` is active |
| Color | `syntax on` |

**Why a sysadmin needs this on RHCSA EX200:** Modest, defaults-friendly `~/.vimrc` — set it up before the exam to feel at home.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `vi` (minimal) doesn't honor `.vimrc` | The mini `vi` ignores it. The full `vim` reads it. |
| Want machine-wide defaults | Edit `/etc/vimrc` (RHEL) |

---

### Task 19 — Edit a systemd unit override the right way

**Purpose:** `systemctl edit` opens an override file in `vi`, with the parent unit shown in a comment. This is the canonical RHCA way.

```bash
sudo systemctl edit sshd
```

Inside the editor (showing a blank `override.conf`):

```
i
[Service]
LimitNOFILE=4096
Esc
:wq
sudo systemctl daemon-reload
sudo systemctl restart sshd
systemctl cat sshd | head -20
```

**Commands**

| Command | Action |
|---|---|
| `sudo systemctl edit UNIT` | Open `/etc/systemd/system/UNIT.d/override.conf` |
| `sudo systemctl edit --full UNIT` | Edit a full copy of the unit (rare; prefer overrides) |
| `sudo systemctl cat UNIT` | Show merged unit definition |
| `daemon-reload` | Reload systemd after a unit change |

**Output decoded**

| Element | Meaning |
|---|---|
| `[Service]` section in override | New directives added |
| `daemon-reload` | systemd re-reads units |
| `restart sshd` | Apply the override |

**Why a sysadmin needs this on RHCA RH358:** The exam-standard service tuning workflow.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `EDITOR` not set | `sudo EDITOR=vim systemctl edit sshd` |
| Override empty after save | Make sure you typed inside Insert mode |

---

### Task 20 — Exam-style scenario: precise edit + verify

**Task statement (RHCSA-style):** *"In `/etc/ssh/sshd_config`, set `PermitRootLogin no`, `PasswordAuthentication no`, and `MaxAuthTries 3`. Preserve a `.bak` backup. Verify syntactically and show the unified diff."*

```bash
# 1. Backup
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 2. Open at the first relevant key
sudo vi +/PermitRootLogin /etc/ssh/sshd_config
```

Inside `vi`:

```
:set nu
/PermitRootLogin
:s/.*PermitRootLogin.*/PermitRootLogin no/      (replace whole line)
/PasswordAuthentication
:s/.*PasswordAuthentication.*/PasswordAuthentication no/
/MaxAuthTries
:s/.*MaxAuthTries.*/MaxAuthTries 3/
:wq
```

Then in the shell:

```bash
sudo sshd -t && echo "syntax OK"
sudo diff -u /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
```

**Expected output (excerpt):**

```
syntax OK
--- /etc/ssh/sshd_config.bak
+++ /etc/ssh/sshd_config
@@ -38,3 +38,3 @@
-#PermitRootLogin prohibit-password
+PermitRootLogin no
…
-#PasswordAuthentication yes
+PasswordAuthentication no
…
-#MaxAuthTries 6
+MaxAuthTries 3
```

**Step-by-step rationale**

| Step | Why |
|---|---|
| `cp /etc/X /etc/X.bak` | Reversible — known-good restore point |
| `vi +/PATTERN` | Land directly on the first edit |
| `:set nu` | Confirm line numbers (helps reproduce edits) |
| Repeated `/SEARCH` + `:s/.*KEY.*/KEY VALUE/` | Whole-line replace removes any `#` comment |
| `:wq` | Save and quit |
| `sshd -t` | Syntax check before any restart |
| `diff -u` | Prove exactly what changed (Lab 23 muscle memory) |

**Output decoded**

| Element | Meaning |
|---|---|
| `syntax OK` | Safe to `systemctl reload sshd` |
| Unified diff | Three settings updated; comments stripped |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `Permission denied` | You forgot `sudo vi` — use Task 17's `:w !sudo tee %` trick |
| `:s` complains "Pattern not found" | The current line doesn't contain the pattern; search first or use `:%s/.*KEY.*/KEY VALUE/` |
| sshd test fails | Restore from backup: `sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config` |

---

## 🔍 vi Decision Guide

```
"I just want to look — not edit"         → view FILE   (or vi -R FILE)
"Edit a config file"                     → vi FILE                       (always backup first)
"Edit a system file"                     → sudoedit FILE   or  sudo vi FILE
"Edit a systemd unit"                    → systemctl edit UNIT
"I broke something"                      → :q!     (or u repeatedly)
"My typing isn't appearing"              → press Esc (you're in Command mode), then i
"I can't save"                           → :w!  (force);  or  :w !sudo tee %
"Jump to line N"                         → vi +N FILE   or  inside: :N
"Find a string"                          → vi +/PATTERN FILE   or  /PATTERN
"Replace everywhere"                     → :%s/A/B/gc           (always with c at first)
"Paste safely"                           → :set paste, paste, :set nopaste
"Recover lost work"                      → choose R when the swap-file dialog appears
```

---

## ✅ Lab Checklist (20 Tasks)

- [ ] 01 Open `vi` and identify the three modes
- [ ] 02 Enter Insert with `i`, `A`, `o`
- [ ] 03 Navigate with `hjkl`, `0`, `$`, `^`, `w`, `b`, `gg`, `G`
- [ ] 04 Delete with `x`, `dd`, `dw`, `D`
- [ ] 05 Yank and paste with `yy`, `Nyy`, `p`, `P`
- [ ] 06 Save and quit with `:w`, `:wq`, `:x`, `ZZ`, `:q!`
- [ ] 07 Search with `/`, `?`, `n`, `N`, `*`, `:noh`
- [ ] 08 Line-scoped substitute `:s/A/B/`
- [ ] 09 Whole-file `:%s/A/B/g[c]`
- [ ] 10 Undo `u`, redo `Ctrl-r`, repeat `.`
- [ ] 11 `:set paste` for safe paste
- [ ] 12 `:e`, `:!CMD`, `:r FILE`, `:r !CMD`
- [ ] 13 Jump to line N (`:N` / `NG`); `:set nu`
- [ ] 14 Open at pattern: `vi +/PATTERN FILE`
- [ ] 15 Indent / dedent with `>>`, `<<`, visual + `>`
- [ ] 16 Recover from a swap file
- [ ] 17 Save with `:w !sudo tee %` when you opened without sudo
- [ ] 18 Build a minimal `~/.vimrc`
- [ ] 19 `systemctl edit UNIT` workflow
- [ ] 20 Exam combo: backup + edit + syntax check + diff

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| Typing in Command mode by accident | Random commands run | Press `Esc`, then `u` to undo |
| Pressing `Q` (uppercase) | Drops into Ex mode (`E173: …`) | Type `visual` and Enter to escape |
| `:wq` without sudo on system file | Permission denied | `:w !sudo tee %` or restart with `sudoedit` |
| Pasting code with auto-indent on | Staircase indentation | `:set paste` before pasting |
| Caps Lock on | All commands feel "wrong" | Toggle Caps Lock — `G` vs `g` etc. matter |
| `Ctrl-S` to save (muscle memory) | Terminal freezes (XOFF) | Press `Ctrl-Q` to unfreeze |
| Mouse over SSH | Stray clicks switch position | `:set mouse=` to disable |
| Opening huge log | Slow / OOM | Use `less` (Lab 20) instead |
| Forget to `:set nopaste` | Subsequent edits skip auto-indent | Toggle off |
| Edited `vi` (minimal) and `.vimrc` is ignored | Settings don't stick | That's expected — full `vim` honors it |

---

## 📌 Exam Strategy

**RHCSA EX200**
- Open every config with `sudo vi +/KEY FILE` to jump directly.
- After editing a system service config, run `sudo SERVICE -t` to syntax-check before restarting.
- Use `:%s/.*KEY.*/KEY VALUE/` to replace a setting whether it was commented or not.

**RHCE EX294 (Ansible)**
- `:set tabstop=2 shiftwidth=2 expandtab` makes YAML painless.
- `:set paste` before pasting playbook fragments.
- `ansible-playbook --syntax-check site.yml` after edits.

**CKA**
- `kubectl edit deploy/X` uses `$EDITOR` (default `vi`). Be ready.
- Set `export EDITOR=vim` at session start.

**RHCA**
- RH342: `:r !journalctl --since '5 min ago' -u UNIT` stamps live logs into runbooks.
- RH358: `systemctl edit` for safe overrides.
- RH236: edit brick config per node — `for h in nodes; do ssh "$h" sudo vi /path/conf; done` (or use Ansible).

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 19 — `cat` | `cat -n` finds the line number to feed `vi +N` |
| Lab 20 — `less` | Press `v` inside `less` to drop straight into `vi` |
| Lab 22 — `grep` | `grep -n` + `vi +N` is the canonical edit-jump pipeline |
| Lab 23 — `diff` / `vimdiff` | The interactive merge tool built on `vi` |
| Lab 24 — `sed` | When edits are scriptable, use `sed -i`; when interactive, use `vi` |
| Lab 27 — `vipw`, `vigr` | Special-purpose `vi` wrappers for `/etc/passwd` and `/etc/group` |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

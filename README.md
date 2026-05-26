# Lab: Command Mode and Insert Mode — `vi` / `vim`

**Series:** linux-ops-mastery — RHCSA Text File Management
**Subjects covered:** the three modes (Normal/Command, Insert, Command-line/Ex), movement (`h j k l`, `w b e`, `0 ^ $`, `gg G`, `Ngg`, `Ctrl-d`/`Ctrl-u`, `Ctrl-f`/`Ctrl-b`), edits (`i a I A o O`, `x X dd dw D yy yw p P r R s S c cc`), search and replace (`/`, `?`, `n N`, `:%s/old/new/gc`), ex commands (`:w`, `:q`, `:wq`, `:q!`, `:x`, `ZZ`, `:e!`, `:set number/nonumber`), visual mode (`v V Ctrl-v`), undo/redo (`u`, `Ctrl-r`), buffers and windows (`:split`, `:vsplit`, `Ctrl-w w/h/j/k/l`), `~/.vimrc` basics, the relationship between `vi`, `vim`, and `nvim`, why RHCSA tests vi specifically, and recovery from `swp` files
**Career arcs covered:** RHCSA (mandatory editor on the exam — no nano), RHCE (Ansible playbook editing on remote hosts), SRE (quick remote config edits over SSH), DevOps (commit messages, Dockerfile edits in CI shells), AI/MLOps (editing training configs on GPU boxes that only have vim)
**Prerequisite:** Lab 05 (filesystem nav), Lab 22 (regex flavors so `:%s///` makes sense)
**Time Estimate:** 40 to 60 minutes
**Difficulty arc:** Task 1 foundation (open → insert → save → quit) · 2 motions and basic edits · 3 search + replace · 4 visual mode + yank/put · 5 split windows + buffers · 6 RHCSA exam-realistic capstone

---

## Objective

Open any text file, navigate without arrows, make precise edits, search and substitute globally, and save — all with the muscle memory the RHCSA exam demands. By the end of this lab `vi` stops being an obstacle and starts being faster than a GUI editor.

The capstone is an exam-realistic prompt: *"In `vi`, open a copy of `/etc/ssh/sshd_config`, jump to the line containing `PermitRootLogin`, change its value to `no`, append a footer comment, save and quit. Then re-open and use a single ex substitute command to comment out every line containing `Banner`."*

> **Lab safety note:** Capstone operates on a copy under `/root/vilab/`. Real `sshd_config` is never modified.

---

## Concept: `vi` Is Modal — The Keys Mean Different Things in Different Modes

```
                Normal/Command Mode  ◄── Esc ─── Insert Mode
                        │
                        │ : / ?  
                        ▼
                Command-line (Ex) Mode
```

- **Normal mode** (default on open): every key is a command. `dd` deletes a line, `yy` yanks (copies), `5gg` jumps to line 5.
- **Insert mode** (`i`, `a`, `o`, …): keys go into the file as text. Press **Esc** to return to Normal.
- **Command-line / Ex mode** (`:`): type `:w`, `:q`, `:%s/old/new/g`, etc. Enter executes, Esc cancels.

> **Why this matters:** Every other GUI editor is permanently in Insert mode. `vi` separates *editing commands* from *typing text*, which is why expert users edit faster in `vi` than anywhere else. The RHCSA exam runs in a GUI VM where `nano` is **not installed** — `vi` is the only editor you can count on.

---

## 📜 Why `vi` Exists — The Story

`vi` was written by **Bill Joy** in 1976 at UC Berkeley. It evolved from `ex` — itself an extension of `ed`, the original Unix line editor. The name `vi` is short for **vi**sual, because unlike `ed`/`ex` (which manipulated lines blind), `vi` *displayed the file* — radical for 1976 on 1200-baud terminals.

Bill Joy designed the keybindings around the **ADM-3A terminal** which had:
- `Esc` where today's Caps Lock sits (cheap to press)
- Arrow keys drawn on `h j k l` (cheap to remember)
- No separate Cursor keys (forced him to invent modal editing)

`vim` ("vi improved") was written by **Bram Moolenaar** in 1991 as a free `vi` clone with multi-undo, syntax highlighting, plugins, scripting, and split windows. On RHEL `/usr/bin/vi` is usually a symlink to `vim` in compatibility mode.

> **The point of the story:** The "weird" keybindings are not arbitrary — they were chosen for a specific terminal in 1976. Once you internalize `hjkl` and Esc-back-to-Normal, every other tool that copies vim (`tmux` copy mode, `less`, `man`, `git` log pager, every IDE's "vim mode") feels familiar.

---

## 👪 The vi Family — Who Lives There

| Editor | Notes |
|---|---|
| `vi` | Original visual editor (1976) |
| `vim` | Vi Improved (1991); de facto on Linux |
| `nvim` (Neovim) | Modern fork; better defaults, Lua plugins |
| `nano` | Beginner-friendly, modeless. **Not on RHCSA** |
| `gedit` / `kate` | GUI editors |
| `emacs` | The other classic; modeless |
| `joe` / `mcedit` | Modeless old-school alternatives |
| `view FILE` | Read-only vim |
| `vimdiff` / `vim -d` | Two-pane diff editor |
| `vimtutor` | Built-in 30-minute interactive tutorial |
| `:!shell-cmd` | Run a shell command from within vim |
| `:r !shell-cmd` | Insert command output into the buffer |

### vi/vim cheat sheet (the high-value subset)

**Movement (Normal mode)**

| Keys | Move to |
|---|---|
| `h j k l` | Left / Down / Up / Right |
| `w` / `b` | Next / previous word start |
| `e` | End of word |
| `0` | Start of line |
| `^` | First non-blank of line |
| `$` | End of line |
| `gg` | Top of file |
| `G` | Bottom of file |
| `Ngg` / `:N` | Line N |
| `Ctrl-d` / `Ctrl-u` | Half-page down/up |
| `Ctrl-f` / `Ctrl-b` | Full-page down/up |
| `%` | Jump to matching `(`, `[`, `{` |
| `*` / `#` | Search word under cursor forward/back |

**Entering insert mode**

| Keys | What it does |
|---|---|
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `I` | Insert at first non-blank of line |
| `A` | Append at end of line |
| `o` | Open new line below |
| `O` | Open new line above |
| `s` | Substitute one character |
| `S` / `cc` | Substitute entire line |
| `R` | Replace mode (overwrite) |
| `Esc` | Back to Normal |

**Edits (Normal mode)**

| Keys | What |
|---|---|
| `x` | Delete char under cursor |
| `X` | Delete char before cursor |
| `dd` | Delete line (cut) |
| `D` | Delete to end of line |
| `dw` | Delete word |
| `d$` | Same as `D` |
| `yy` / `Y` | Yank (copy) line |
| `yw` | Yank word |
| `p` / `P` | Paste after / before |
| `r<char>` | Replace single character |
| `u` | Undo |
| `Ctrl-r` | Redo |
| `.` | Repeat last change |
| `J` | Join next line into this one |
| `>>` / `<<` | Indent / outdent line |

**Search and replace**

| Keys | What |
|---|---|
| `/pattern` | Search forward (regex) |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | …with confirm per match |
| `:7,20s/old/new/g` | …only lines 7–20 |
| `:g/PAT/d` | Delete every line matching PAT |
| `:v/PAT/d` | Delete every line NOT matching PAT |

**Ex command-line basics**

| Command | What |
|---|---|
| `:w` | Write (save) |
| `:w FILE` | Save as |
| `:q` | Quit |
| `:q!` | Quit, discard changes |
| `:wq` / `ZZ` | Save + quit |
| `:x` | Same as `:wq` (no write if unchanged) |
| `:e FILE` | Edit FILE in current window |
| `:e!` | Re-read current file from disk (discard) |
| `:r FILE` | Read FILE into buffer at cursor |
| `:r !cmd` | Read shell command output |
| `:!cmd` | Run shell command (output shown briefly) |
| `:set number` / `:set nonumber` | Toggle line numbers |
| `:set ignorecase` / `:set noignorecase` | Search case sensitivity |
| `:set paste` / `:set nopaste` | Paste-friendly mode (no auto-indent) |
| `:help TOPIC` | Built-in help |

**Visual mode**

| Keys | What |
|---|---|
| `v` | Char-wise visual select |
| `V` | Line-wise visual select |
| `Ctrl-v` | Block (column) visual select |
| `y` / `d` / `c` / `>` / `<` | Yank / delete / change / indent / outdent the selection |

> **The point of the family tree:** You will use `i a o`, `hjkl`, `dd yy p`, `gg G`, `/pattern`, `:%s///g`, `:w`, `:q` more than the entire rest of vim combined.

---

## 🔬 The Anatomy of `vim sshd_config` — In One Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ # /etc/ssh/sshd_config                                       │  ← buffer content
│ #                                                            │
│ PermitRootLogin no█                                          │  ← cursor (█) — Normal mode
│ Port 22                                                      │
│ ...                                                          │
│ ~                                                            │  ← `~` means "past end of file"
│ ~                                                            │
├─────────────────────────────────────────────────────────────┤
│ sshd_config            38,18    Top                          │  ← status line (filename, line,col, position)
│ :wq█                                                         │  ← Ex command line
└─────────────────────────────────────────────────────────────┘
       └─ When you type `:` the cursor jumps here.
```

> **Reading rule:** If your typed letters appear in the buffer, you're in Insert mode (press Esc). If they don't, you're in Normal mode. If they appear at the bottom after `:` or `/`, you're in Ex/search mode.

---

## 📚 vi Reference Table

| Task | Keys |
|---|---|
| Open file | `vi FILE` |
| Open at line N | `vi +N FILE` |
| Open at first match | `vi +/PAT FILE` |
| Open read-only | `view FILE` |
| Save | `:w` |
| Save + quit | `:wq` or `ZZ` |
| Quit without save | `:q!` |
| Quit if no changes | `:q` |
| Save as | `:w newname` |
| Reload from disk | `:e!` |
| Show line numbers | `:set number` |
| Jump to line N | `:N` or `Ngg` |
| Find next | `/PAT` then `n` |
| Find previous | `?PAT` then `n` |
| Replace all | `:%s/OLD/NEW/g` |
| Replace with confirm | `:%s/OLD/NEW/gc` |
| Replace in range | `:N,Ms/OLD/NEW/g` |
| Delete lines matching | `:g/PAT/d` |
| Indent / outdent | `>>` / `<<` |
| Undo / redo | `u` / `Ctrl-r` |
| Copy/cut line | `yy` / `dd` |
| Paste | `p` / `P` |
| Repeat last change | `.` |
| Split windows | `:split` / `:vsplit` |
| Switch window | `Ctrl-w w` (or h/j/k/l) |
| Open another file | `:e otherfile` |
| List buffers | `:ls` |
| Switch buffer | `:bN` |
| Run shell command | `:!cmd` |
| Insert command output | `:r !cmd` |

> **Rule one of vi:** Press **Esc** any time you don't know what mode you're in. From Normal mode every other command works.

---

## 🎯 Career Pathway Sidebar

| Level | Why this lab matters |
|---|---|
| **RHCSA candidate** | The exam VM has `vi`/`vim` and not always `nano`. Editing fluently in `vi` saves you 10–15 minutes across the exam. |
| **RHCE candidate** | Editing playbooks/inventory in `vi` is standard. Re-indenting blocks (`>>`) and searching with `/` are constant. |
| **SRE / Platform** | SSH into a host with no IDE: `vi /etc/whatever`. Often the only editor installed on minimal images. |
| **DevOps** | `git commit` opens `$EDITOR` — usually `vi`. Splitting Dockerfiles into stages with `O` and visual mode is fast. |
| **AI / MLOps** | GPU servers and training nodes are minimal Linux installs — `vim` plus a `.vimrc` is your IDE. |

---

## 🔧 The 6 Tasks

> Six exam-realistic phases that build **open → insert → motion → search-replace → visual → capstone** muscle memory.

---

### Task 1 — Open, insert, save, quit

```bash
mkdir -p ~/vilab && cd ~/vilab
vi hello.txt
# Inside vi:
#   i             enter Insert mode
#   Hello, $USER  type some text
#   <Enter>
#   This was written in vi.
#   Esc           back to Normal
#   :wq           save and quit

cat hello.txt
```

---

### Task 2 — Motions + basic edits

```bash
cp /etc/services ~/vilab/services.txt
vi ~/vilab/services.txt
# Inside vi:
#   :set number   show line numbers
#   gg            go to top
#   G             go to bottom
#   50gg          jump to line 50
#   /ssh          search "ssh"; press n/N to step
#   *             search the word under cursor
#   dd            delete current line
#   u             undo
#   yy p          duplicate the current line
#   .             repeat the last change
#   :q!           quit without saving
```

---

### Task 3 — Search + replace (single file)

```bash
cp /etc/ssh/sshd_config ~/vilab/sshd_config
vi ~/vilab/sshd_config
# Inside vi:
#   /PermitRootLogin     find it
#   ciw                  change inner word (the value) → enter insert mode for the value
#   no                   type the new value
#   Esc
#   :%s/^#Port 22/Port 22/g    uncomment Port 22 globally
#   :%s/^Banner /#Banner /gc   comment Banner with confirm
#   :w                   save
#   :q                   quit
```

---

### Task 4 — Visual mode (line + block)

```bash
vi ~/vilab/sshd_config
# Inside vi:
#   gg                   top
#   V                    line visual
#   10j                  extend selection 10 lines down
#   >                    indent the block
#   u                    undo
#   Ctrl-v               block (column) visual
#   3l5j                 select a small rectangle
#   r#                   replace each char in the block with '#'
#   u                    undo
#   y                    yank the rectangle
#   p                    paste it
#   :wq
```

---

### Task 5 — Splits, buffers, ex globals

```bash
vi ~/vilab/sshd_config
# Inside vi:
#   :vsplit ~/vilab/services.txt   vertical split with another file
#   Ctrl-w l                       move to right window
#   Ctrl-w h                       move back to left window
#   :ls                            list buffers
#   :b 2                           jump to buffer 2
#   :g/^#/d                        delete every comment line (current buffer)
#   :v/^$/d                        delete every NON-blank line — (don't run on this file!)
#   :e!                            discard changes, re-read from disk
#   :qa!                           quit all without saving
```

---

### Task 6 — Capstone: harden `sshd_config` in vi only

**Task statement:** Working on a copy at `/root/vilab/sshd_config`, use only `vi` commands to:
1. Toggle line numbers on.
2. Replace any existing `PermitRootLogin` setting with `PermitRootLogin no`.
3. Append a footer comment `# hardened on <date>` at the last line.
4. Re-open and use a single ex command to comment every line containing `Banner`.
5. Save and quit cleanly. Use `diff` against the original to prove the change.

```bash
sudo -i
mkdir -p /root/vilab
cp -a /etc/ssh/sshd_config /root/vilab/sshd_config
cp -a /etc/ssh/sshd_config /root/vilab/sshd_config.orig

vi /root/vilab/sshd_config
# Inside vi:
#   :set number
#   /PermitRootLogin               find directive
#   V                              visual line
#   c                              change the entire line
#   PermitRootLogin no             type the replacement
#   Esc
#   G                              go to last line
#   o                              open new line below
#   # hardened on <Ctrl-r =strftime("%F")<Enter>>     (or just type a date string)
#   Esc
#   :wq

# Re-open and comment Banner via single ex global
vi /root/vilab/sshd_config
# Inside vi:
#   :g/^Banner /s//#&/
#   :wq

# Prove the diff
diff -u /root/vilab/sshd_config.orig /root/vilab/sshd_config | head -n 40
```

**Expected verification output (excerpt):**

```text
--- /root/vilab/sshd_config.orig    ...
+++ /root/vilab/sshd_config         ...
@@ -38,1 +38,1 @@
-#PermitRootLogin prohibit-password
+PermitRootLogin no
@@ -86,1 +86,1 @@
-Banner /etc/issue.net
+#Banner /etc/issue.net
@@ -120,0 +121,1 @@
+# hardened on 2026-05-26
```

**Cleanup**

```bash
rm -rf /root/vilab
exit
```

---

## 🔍 vi Decision Guide

```
What do you want to do?
  ├── "Insert text"                              → i / a / o / O   then type   then Esc
  ├── "Move without arrows"                      → h j k l, w b e, 0 ^ $, gg G
  ├── "Jump to line N"                           → :N   or   Ngg
  ├── "Find / find next"                         → /PAT   then   n / N
  ├── "Replace once / everywhere / w/ confirm"   → :s/old/new/   :%s/old/new/g   :%s/old/new/gc
  ├── "Delete every comment line"                → :g/^#/d
  ├── "Save / quit / both / discard"             → :w   :q   :wq   :q!
  ├── "Open another file beside it"              → :vsplit other  / :split other
  ├── "Run a shell command"                      → :!cmd   /   :r !cmd  (insert output)
  ├── "I'm lost — what mode am I in?"            → press Esc; you're now in Normal mode
  └── "Show line numbers"                        → :set number   (`:set nu` short form)
```

---

## ✅ Lab Checklist (6 Tasks)

- [ ] 01 Create a file, insert, save, quit
- [ ] 02 Motions (`gg`, `G`, `Ngg`, `/PAT`) and basic edits (`dd`, `yy`, `p`, `u`, `.`)
- [ ] 03 Search + `:%s/old/new/gc` substitution
- [ ] 04 Visual mode line and block selection
- [ ] 05 Splits and ex globals (`:g/PAT/d`)
- [ ] 06 Capstone — harden `sshd_config` in vi and verify with diff

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| Typed `:wq` while in Insert mode | `:wq` appears in the file | Esc first, then `:wq` |
| Pressed Caps Lock | "Why is every command broken?" | Turn off Caps Lock |
| Used arrow keys and never learned `hjkl` | Slow long-term | Force yourself for one week |
| Yanked, came back to shell, paste was wrong | `yy` yanks to vim's `"` register, not the clipboard | `"+yy` (system clipboard) requires vim with `+clipboard` |
| Lost an unsaved swap file | Crashed mid-edit | `vim FILE` will offer to recover from `.FILE.swp` |
| `:q` refused to quit | Unsaved changes | `:w` to save or `:q!` to discard |
| Paste from terminal added auto-indent staircase | Paste mode off | `:set paste`, paste, `:set nopaste` |
| `:%s/old/new/` only replaced first per line | Forgot `g` flag | `:%s/old/new/g` |
| `:%s/path/old/new/` broke | `/` inside pattern | Use a different delimiter `:%s|/etc/old|/etc/new|g` |
| Cannot exit vim (the famous joke) | Permanently in some mode | Esc, then `:q!` — always works |

---

## 🎯 Career & Interview Strategy

**RHCSA candidate**
- Run `vimtutor` once end-to-end. Then do every config edit in this curriculum in `vi` (not nano) until it's reflexive.

**RHCE candidate**
- `gg=G` (re-indent the whole file) is gold for YAML playbooks.

**SRE / Platform interview**
- "How would you correlate two log files quickly?" → `vi -O log1 log2`, `Ctrl-w w` to switch, `/error` in each pane, jump between with `:bn`/`:bp`.

**DevOps**
- Author a tiny `~/.vimrc` (line numbers, syntax on, 2-space indent, `set paste` shortcut). Drop it on every server you SSH to.

**AI / MLOps**
- Edit YAML/JSON configs on training nodes; learn `=` (indent), `gqip` (reflow paragraph), `gv` (re-select last visual).

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 19 — `cat` | Read first, edit only when needed |
| Lab 22 — `grep` | Same regex flavor |
| Lab 24 — `sed` | Non-interactive cousin of `:%s///g` |
| Lab 23 — `diff` | Compare before/after vi edits |
| Lab 27 — `vipw`/`vigr` | Safer wrappers for `/etc/passwd` and `/etc/group` |

---

## 👤 Author

**Kelvin R. Tobias**
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

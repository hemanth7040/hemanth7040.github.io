---
title: "Vim Commands for DevOps Engineers"
date: 2026-05-07
draft: false
categories: ["vim"]
description: "A complete guide to essential Vim commands every DevOps engineer must know — with examples for every command"
tags: ["devops", "linux", "cheatsheet"]
---

# Vim Commands for DevOps Engineers

Vim is a powerful text editor available on almost every Linux/Unix system. As a DevOps engineer,
you will use it daily to edit config files, YAML pipelines, Dockerfiles, and shell scripts —
often on remote servers where no GUI is available. This guide covers all the essential commands
with simple explanations and a real example for every single command.

---

## Modes

Vim has different modes. You need to switch between them to do different things.

| Command | Description |
|---------|-------------|
| `i` | Insert mode before the cursor |
| `a` | Insert mode after the cursor |
| `I` | Insert at the beginning of the line |
| `A` | Insert at the end of the line |
| `v` | Visual mode (select character by character) |
| `V` | Visual line mode (select entire lines) |
| `Ctrl+v` | Visual block mode (column selection) |
| `Esc` | Return to Normal mode |
---

### `i` — Insert mode before the cursor

Start typing at the current cursor position.

```
# Your cursor is here → [l]isten: 80
# You want to add "# " before "listen"
Press i
Type: # 
Result: # listen: 80
```

---

### `a` — Insert mode after the cursor

Start typing one character after where the cursor is.

```
# Your cursor is on the last digit → listen: 8[0]
# You want to change it to 8080
Press a
Type: 80
Result: listen: 8080
```

---

### `I` — Insert at the beginning of the line

Jumps to the very start of the line and enters insert mode.

```
# You are anywhere on this line:
    server_name example.com;
# You want to add a comment at the very start
Press I
Type: # 
Result: #     server_name example.com;
```

---

### `A` — Insert at the end of the line

Jumps to the very end of the line and enters insert mode.

```
# Your line looks like this:
EXPOSE 8080
# You want to add a comment after it
Press A
Type:   # app port
Result: EXPOSE 8080   # app port
```

---

### `v` — Visual mode (select character by character)

Lets you highlight text one character at a time.

```
# You want to select and delete just the word "debug"
# in this line: log_level: debug
Move cursor to "d" in debug
Press v
Press w        → selects the word "debug"
Press d        → deletes it
Result: log_level: 
```

---

### `V` — Visual line mode (select entire lines)

Selects one full line at a time.

```
# You want to copy 3 lines of a docker-compose service block
Press V
Press 2j       → selects 3 lines downward
Press y        → copies all 3 lines
Move to where you want to paste
Press p        → pastes them
```

---

### `Ctrl+v` — Visual block mode (column selection)

Selects a rectangular block across multiple lines — perfect for adding # to many lines at once.

```
# You want to comment out these 4 lines:
RUN apt-get update
RUN apt-get install curl
RUN apt-get install git
RUN apt-get clean

Press Ctrl+v
Press 3j       → select 4 lines downward (column)
Press I        → insert at start of block
Type: # 
Press Esc      → # is added to all 4 lines

Result:
# RUN apt-get update
# RUN apt-get install curl
# RUN apt-get install git
# RUN apt-get clean
```

---

### `Esc` — Return to Normal mode

Exit insert or visual mode and go back to normal mode.

```
# You were typing and want to stop
i              → entered insert mode
Type: server_name myapp.com;
Press Esc      → back to normal mode, safe to run other commands
```

---

## File Operations

| Command | Description |
|---------|-------------|
| `vim filename` | Open a file |
| `:w` | Save the file |
| `:q` | Quit Vim |
| `:wq` / `:x` / `ZZ` | Save and quit |
| `:q!` | Force quit without saving |
| `:wq!` | Force save and quit |
| `:e filename` | Open another file |
| `:w filename` | Save as a new file |
---

### `vim filename` — Open a file

Opens a file in Vim from the terminal.

```
vim /etc/nginx/nginx.conf
vim Dockerfile
vim deployment.yaml
```

---

### `:w` — Save the file

Writes (saves) your changes to disk without closing Vim.

```
# You edited the nginx config and want to save
:w
# Bottom of screen shows: "nginx.conf" 42L, 1.2K written
```

---

### `:q` — Quit Vim

Closes Vim. Only works if you have no unsaved changes.

```
# You opened a file just to read it, no changes made
:q
# Vim closes cleanly
```

---

### `:wq` or `:x` or `ZZ` — Save and quit

Saves your changes and closes Vim in one step.

```
# You finished editing the Dockerfile
:wq
# OR
:x
# OR (in normal mode, no colon needed)
ZZ
```

---

### `:q!` — Force quit without saving

Exits Vim and throws away all changes. Use when you made a mistake and want to start over.

```
# You accidentally edited the wrong file
:q!
# Vim exits, original file is untouched
```

---

### `:wq!` — Force save and quit

Saves even if the file is read-only (if you have sudo permission).

```
# File is owned by root but you opened with sudo vim
sudo vim /etc/hosts
# Make changes
:wq!
# Forces the save
```

---

### `:e filename` — Open another file

Opens a different file without leaving Vim.

```
# You are editing nginx.conf and want to check the upstream config
:e /etc/nginx/conf.d/upstream.conf
# Switches to that file in the same Vim window
```

---

### `:w filename` — Save as a new file

Saves a copy of the current content to a different file name.

```
# You want to back up the config before making changes
:w /etc/nginx/nginx.conf.bak
# Original file stays open, backup is saved
```

---

## Navigation

| Command | Description |
|---------|-------------|
| `h j k l` | Move left / down / up / right |
| `gg` | Go to the first line |
| `G` | Go to the last line |
| `:42` | Go to a specific line number |
| `0` | Go to the start of the line |
| `$` | Go to the end of the line |
| `w` | Jump to the next word |
| `b` | Jump to the previous word |
| `Ctrl+d` | Scroll half page down |
| `Ctrl+u` | Scroll half page up |
| `Ctrl+f` | Scroll full page down |
| `Ctrl+b` | Scroll full page up |

---

### `h j k l` — Move left / down / up / right

The basic movement keys in Vim (instead of arrow keys).

```
h → move left  (like ←)
j → move down  (like ↓)
k → move up    (like ↑)
l → move right (like →)

# Move down 5 lines quickly
5j
# Move right 3 characters
3l
```

---

### `gg` — Go to the first line

Jumps straight to line 1 of the file.

```
# You are at the bottom of a 500-line config file
# and want to go back to the top
gg
# Cursor jumps to line 1 instantly
```

---

### `G` — Go to the last line

Jumps to the very last line of the file.

```
# You want to add a new rule at the end of a firewall script
G
# Cursor is now on the last line
A          → start typing at end of that line
```

---

### `:42` — Go to a specific line number

Jumps directly to any line by its number.

```
# Your CI pipeline log says "error on line 87" in your Jenkinsfile
:87
# Cursor jumps to line 87 directly
```

---

### `0` — Go to the start of the line

Moves the cursor to the very first character of the current line.

```
# You are at the end of this line:
    listen       80;
# and want to go to the start to add a comment
Press 0
# Cursor is now on the first character of the line
```

---

### `$` — Go to the end of the line

Moves the cursor to the last character of the current line.

```
# You want to append a semicolon at the end of a line that is missing one
Press $
Press a        → insert after last character
Type: ;
Press Esc
```

---

### `w` — Jump to the next word

Moves the cursor forward one word at a time.

```
# Line: server_name example.com myapp.com;
# Cursor is on "server_name", press w to move word by word:
w → cursor on "example.com"
w → cursor on "myapp.com"
w → cursor on ";"
```

---

### `b` — Jump to the previous word

Moves the cursor backward one word at a time.

```
# Line: server_name example.com myapp.com;
# Cursor is on "myapp.com", press b to go back:
b → cursor on "example.com"
b → cursor on "server_name"
```

---

### `Ctrl+d` — Scroll half page down

Moves the view down by half a screen — useful for long config files.

```
# You are reading a long /etc/ssh/sshd_config
# and want to scroll through it quickly
Ctrl+d     → moves half page down
Ctrl+d     → moves another half page down
```

---

### `Ctrl+u` — Scroll half page up

Moves the view up by half a screen.

```
# You scrolled too far down, go back up
Ctrl+u     → moves half page up
```

---

### `Ctrl+f` — Scroll full page down

Moves the view down by a full screen.

```
# Quickly jump through a very long Terraform file
Ctrl+f     → full page down
Ctrl+f     → another full page down
```

---

### `Ctrl+b` — Scroll full page up

Moves the view up by a full screen.

```
# Go back to the top area of the file quickly
Ctrl+b     → full page up
```

---

## Editing

| Command | Description |
|---------|-------------|
| `dd` | Delete (cut) the current line |
| `3dd` | Delete multiple lines at once |
| `yy` | Copy (yank) the current line |
| `3yy` | Copy multiple lines |
| `p` | Paste after the cursor |
| `P` | Paste before the cursor |
| `x` | Delete one character |
| `dw` | Delete a word |
| `D` | Delete to end of line |
| `cc` | Delete line and start typing |
| `cw` | Change a word |
| `r` | Replace one character |
| `u` | Undo |
| `Ctrl+r` | Redo |
| `.` | Repeat last command |

---

### `dd` — Delete (cut) the current line

Removes the entire line your cursor is on. It is also copied, so you can paste it.

```
# You want to remove a wrong environment variable line
# ENV DEBUG=true   ← cursor is here
dd
# That line is gone (and copied to clipboard)
```

---

### `3dd` — Delete multiple lines at once

Deletes the current line plus the next lines (3 total in this case).

```
# You want to remove an entire 3-line stanza in nginx.conf:
location /old-path {      ← cursor here
    return 301 /new-path;
}
3dd
# All 3 lines are deleted
```

---

### `yy` — Copy (yank) the current line

Copies the current line without deleting it.

```
# You want to duplicate a LABEL line in a Dockerfile
# LABEL maintainer="devops@company.com"   ← cursor here
yy
j          → move down one line
p          → paste the copied line below
```

---

### `3yy` — Copy multiple lines

Copies the current line and the next lines (3 total).

```
# You want to copy a 3-line environment block:
- name: DB_HOST       ← cursor here
  value: "localhost"
- name: DB_PORT
3yy
G          → go to end of file
p          → paste the 3 lines
```

---

### `p` — Paste after the cursor

Pastes whatever was last copied or deleted, below the current line.

```
# You copied a line with yy and want to paste it
yy         → copy current line
5j         → move 5 lines down
p          → paste the copied line below cursor position
```

---

### `P` — Paste before the cursor

Pastes above the current line instead of below.

```
# You want to paste a line ABOVE where you are
yy         → copy a line
k          → move up one line
P          → paste above current line
```

---

### `x` — Delete one character

Deletes only the single character under the cursor.

```
# You have a typo: "poort: 8080" (extra 'o')
# Move cursor to the second 'o' in "poort"
x
# Result: "port: 8080"
```

---

### `dw` — Delete a word

Deletes from the cursor position to the start of the next word.

```
# Line: server_name old-server.com;
# Cursor is on "old-server.com"
dw
# Result: server_name ;
# Now type the new name:
i → new-server.com
```

---

### `D` — Delete to end of line

Deletes everything from the cursor to the end of the line.

```
# Line: listen 80; # TODO: change to 443 later
# Cursor is on "#"
# You want to remove the comment part
D
# Result: listen 80;
```

---

### `cc` — Delete line and start typing

Clears the entire line and puts you in insert mode to type a replacement.

```
# You want to completely rewrite this line:
# server_name _;
# Cursor is anywhere on that line
cc
# Line is cleared, you are now in insert mode
Type: server_name myapp.com;
Press Esc
```

---

### `cw` — Change a word

Deletes the current word and puts you in insert mode to type a new one.

```
# Line: image: nginx:1.19
# Cursor is on "1.19" and you want to update the version
cw
# "1.19" is deleted, insert mode starts
Type: 1.25
Press Esc
# Result: image: nginx:1.25
```

---

### `r` — Replace one character

Replaces just the single character under the cursor without entering insert mode.

```
# Line: port: 8o80   (letter 'o' instead of '0')
# Move cursor to the 'o'
r
Type: 0
# Result: port: 8080
# No need to press Esc — stays in normal mode
```

---

### `u` — Undo

Reverses your last action. Press multiple times to keep undoing.

```
# You accidentally deleted an important line
dd
u
# The line is back!

# Undo multiple steps
u      → undo last change
u      → undo the change before that
u      → keep going back
```

---

### `Ctrl+r` — Redo

Re-applies something you undid.

```
# You undid a change but actually it was correct
u          → undid a change
Ctrl+r     → redid it, change is back
```

---

### `.` — Repeat last command

Repeats whatever editing command you last ran — very powerful for repetitive edits.

```
# You deleted a line with dd and want to delete more lines too
dd         → deleted line 1
.          → deletes the next line (repeats dd)
.          → deletes another line

# You changed a word with cw and want to change all similar words
cw         → changed word to new value
n          → jump to next match
.          → applies the same change to that word
```

---

## Search & Replace / File-wide Operations

| Command | Description |
|---------|-------------|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Jump to next match |
| `N` | Jump to previous match |
| `*` | Search for word under cursor (forward) |
| `#` | Search for word under cursor (backward) |
| `:%s/old/new/g` | Replace all occurrences |
| `:%s/old/new/gc` | Replace with confirmation |
| `:n,ms/old/new/g` | Replace between specific lines |
| `:%d` | Delete all lines in the file |
| `:%y` | Copy entire file content |
| `:%s/^#.*//g` | Remove all comment lines |
| `:%s/  */ /g` | Replace multiple spaces with one |

---

### `/pattern` — Search forward

Searches for text from the cursor downward.

```
# Find all occurrences of "timeout" in nginx.conf
/timeout
# First match is highlighted, press n to jump to the next one
```

---

### `?pattern` — Search backward

Searches for text from the cursor upward.

```
# You are at the bottom of the file and want to search upward
?server_name
# Finds the previous occurrence above the cursor
```

---

### `n` — Jump to next match

After a search, jump to the next occurrence.

```
/ENV       → finds first ENV in Dockerfile
n          → jumps to second ENV
n          → jumps to third ENV
```

---

### `N` — Jump to previous match

After a search, go back to the previous occurrence.

```
/ENV       → found a match
n          → went forward too far
N          → goes back to previous match
```

---

### `*` — Search for word under cursor (forward)

Place your cursor on any word and press `*` to find all matching words going forward.

```
# Your cursor is on the word "backend" in a config file
*
# Jumps to the next place "backend" appears in the file
n          → keeps jumping forward through all matches
```

---

### `#` — Search for word under cursor (backward)

Same as `*` but searches backward (upward in the file).

```
# Cursor is on "upstream"
#
# Jumps to the previous occurrence of "upstream" above
```

---

### `:%s/old/new/g` — Replace all occurrences

Replaces every instance of a word throughout the entire file.

```
# Update the database host across the whole config
:%s/db-server-old/db-server-new/g
# Every occurrence is replaced instantly

# Update an old IP address everywhere in the file
:%s/192.168.1.100/10.0.0.50/g
```

---

### `:%s/old/new/gc` — Replace with confirmation

Same as above but asks "replace? (y/n)" for each match — safe for important files.

```
# You want to rename a variable but review each change
:%s/api_key/auth_token/gc
# Vim shows each match and asks:
# replace with auth_token (y/n/a/q/l)?
# y = yes this one, n = skip, a = replace all, q = quit
```

---

### `:n,ms/old/new/g` — Replace between specific lines

Replaces text only within a range of lines, not the whole file.

```
# Replace "http" with "https" only between lines 10 and 30
:10,30s/http/https/g
# Only those lines are affected, rest of file is untouched
```

---

### `:%d` — Delete all lines in the file

Wipes out every single line in the buffer. Be very careful with this.

```
# You want to completely rewrite a config file from scratch
:%d
# File buffer is now empty, start typing fresh

# Made a mistake? Undo immediately!
u
# All lines are restored
```

---

### `:%y` — Copy entire file content

Yanks (copies) all lines into the clipboard buffer.

```
# You want to copy everything from one config and paste into another
:%y
:e /etc/nginx/conf.d/new-site.conf
gg         → go to top of new file
p          → paste everything from the old file
```

---

### `:%s/^#.*//g` — Remove all comment lines

Deletes the content of all lines that start with `#`.

```
# Your shell script has many comment lines and you want a clean version
:%s/^#.*//g
# All comment lines are now blank
```

---

### `:%s/  */ /g` — Replace multiple spaces with one

Cleans up files that have inconsistent spacing.

```
# Your file has lines like:
# server_name    example.com     myapp.com;
:%s/  */ /g
# Result:
# server_name example.com myapp.com;
```

---

## Vim Settings

| Command | Description |
|---------|-------------|
| `:set number` | Show line numbers |
| `:set nonumber` | Hide line numbers |
| `:set paste` | Enable paste mode |
| `:set nopaste` | Disable paste mode |
| `:set tabstop=2` | Set tab width |
| `:set expandtab` | Use spaces instead of tabs |
| `:syntax on` | Enable syntax highlighting |
| `:set hlsearch` | Highlight search results |

---

### `:set number` — Show line numbers

Displays line numbers on the left side of the screen.

```
:set number
# Now every line has a number:
# 1  server {
# 2      listen 80;
# 3      server_name example.com;
```

---

### `:set nonumber` — Hide line numbers

Turns off line numbers.

```
:set nonumber
# Line numbers disappear from the left side
```

---

### `:set paste` — Enable paste mode

Prevents Vim from auto-indenting or reformatting when you paste from clipboard.

```
# Without paste mode, pasting YAML from clipboard can break indentation
:set paste
# Now paste with Ctrl+Shift+V — indentation is preserved exactly as copied
```

---

### `:set nopaste` — Disable paste mode

Turn off paste mode after you are done pasting so normal features work again.

```
:set nopaste
# Auto-indent and other features work normally again
```

---

### `:set tabstop=2` — Set tab width

Controls how wide a tab character appears on screen.

```
# For Kubernetes YAML files (2-space indent is standard)
:set tabstop=2
# Tabs now display as 2 spaces wide
```

---

### `:set expandtab` — Use spaces instead of tabs

When you press Tab, Vim inserts spaces instead of a tab character.

```
# Critical for YAML files — tab characters cause parse errors in YAML!
:set expandtab
:set tabstop=2
# Now the Tab key inserts 2 spaces instead of a tab character
```

---

### `:syntax on` — Enable syntax highlighting

Colors your code based on the file type — makes config files much easier to read.

```
vim Dockerfile
:syntax on
# Keywords like FROM, RUN, COPY are now highlighted in different colors
```

---

### `:set hlsearch` — Highlight search results

After you search, all matching words are highlighted across the file.

```
:set hlsearch
/listen
# Every occurrence of "listen" is highlighted in yellow across the file
```

---

## Split Windows & Tabs

| Command | Description |
|---------|-------------|
| `:sp filename` | Horizontal split |
| `:vsp filename` | Vertical split |
| `Ctrl+w w` | Switch between splits |
| `Ctrl+w q` | Close a split |
| `:tabnew filename` | Open file in a new tab |
| `gt` | Go to next tab |
| `gT` | Go to previous tab |

---

### `:sp filename` — Horizontal split

Opens a second file in a pane above/below the current one.

```
# You want to check a reference config while editing a new one
:sp /etc/nginx/nginx.conf.example
# Screen splits horizontally — new file on top, current file on bottom
Ctrl+w w    → switch between the two panes
```

---

### `:vsp filename` — Vertical split

Opens a second file in a pane side-by-side with the current one.

```
# Compare current deployment.yaml with the previous version side by side
:vsp deployment.yaml.bak
# Two files shown side by side for easy comparison
Ctrl+w w    → switch focus between left and right pane
```

---

### `Ctrl+w w` — Switch between splits

Moves your cursor/focus to the next open split pane.

```
# You have two split panes open
Ctrl+w w    → focus moves to the other pane
Ctrl+w w    → focus moves back to original pane
```

---

### `Ctrl+w q` — Close a split

Closes the currently focused split pane.

```
# You are done looking at the reference file in the split
Ctrl+w q
# That split closes, you are back to a single window
```

---

### `:tabnew filename` — Open file in a new tab

Opens a file in a new Vim tab (similar to browser tabs).

```
# You are editing nginx.conf and want to open the SSL config separately
:tabnew /etc/nginx/conf.d/ssl.conf
# A new tab opens with the SSL config
# Tab bar at the top shows: nginx.conf | ssl.conf
```

---

### `gt` — Go to next tab

Switches to the tab on the right.

```
# You have 3 tabs open: nginx.conf | ssl.conf | upstream.conf
# Currently on nginx.conf
gt    → switches to ssl.conf
gt    → switches to upstream.conf
```

---

### `gT` — Go to previous tab

Switches to the tab on the left.

```
# Currently on upstream.conf
gT    → switches to ssl.conf
gT    → switches back to nginx.conf
```

---

## Comment / Uncomment Multiple Lines

| Command | Description |
|---------|-------------|
| `Ctrl+v` → `I` → `#` → `Esc` | Comment out multiple lines |
| `Ctrl+v` → select `#` column → `d` | Uncomment multiple lines |

---

### Comment out multiple lines using `Ctrl+v`

```
# You want to disable these 3 lines in a shell script:
export DB_HOST=localhost       ← move cursor here first
export DB_PORT=5432
export DB_NAME=myapp

Step 1: Press Ctrl+v             → enter visual block mode
Step 2: Press 2j                 → select all 3 lines downward
Step 3: Press I                  → insert at beginning of block
Step 4: Type: #                  → type the comment character
Step 5: Press Esc                → # is added to all 3 lines at once

Result:
#export DB_HOST=localhost
#export DB_PORT=5432
#export DB_NAME=myapp
```

---

### Uncomment multiple lines using `Ctrl+v`

```
# You want to re-enable those commented lines:
#export DB_HOST=localhost       ← move cursor onto the # here
#export DB_PORT=5432
#export DB_NAME=myapp

Step 1: Press Ctrl+v             → enter visual block mode
Step 2: Press 2j                 → select the # column on all 3 lines
Step 3: Press d                  → delete that column

Result:
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=myapp
```

---

## ⚠️ Important Warning About `:%d`

> If you run `:%d` and then save with `:w`, the file on disk becomes permanently empty.
> Always undo with `u` **before** saving!

```
:%d        → all lines deleted from buffer
u          → all lines restored ✅ (safe — file on disk is untouched)

# DANGEROUS combination:
:%d
:w         → file is now empty on disk ❌ (hard to recover!)
```

---

## Quick Reference — Most Used by DevOps

| Task | Command |
|------|---------|
| Open a file | `vim filename` |
| Save and exit | `:wq` or `ZZ` |
| Exit without saving | `:q!` |
| Go to a specific line | `:87` |
| Search for text | `/text` then `n` |
| Replace text everywhere | `:%s/old/new/g` |
| Delete a line | `dd` |
| Undo | `u` |
| Redo | `Ctrl+r` |
| Delete all lines | `:%d` |
| Undo delete all | `u` |
| Enable line numbers | `:set number` |
| Safe paste (no indent issues) | `:set paste` |
| Comment multiple lines | `Ctrl+v` → select → `I` → `#` → `Esc` |

---

## Pro Tips

- Always use `:set paste` before pasting from your clipboard — it prevents broken indentation.
- Use `ggdG` as a shortcut to delete all content (go to top, then delete to bottom).
- Vim is almost always available on remote servers, even when `nano` is not installed.
- Save your preferred settings in `~/.vimrc` so they load automatically every time you open Vim.

```bash
# Recommended ~/.vimrc for DevOps work
set number
set expandtab
set tabstop=2
set hlsearch
syntax on
```
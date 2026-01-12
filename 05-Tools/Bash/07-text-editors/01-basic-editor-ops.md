---
tags: ['editors', 'vim', 'nano', 'emacs', 'linux', 'tools', 'roadmap']
---

# Basic Editor Operations

## Summary

Text editors are essential tools for working in Unix/Linux systems. Understanding basic editor operations—opening, saving, searching, replacing, and quitting—applies across different editors. While syntax varies between editors (vi, nano, emacs), core concepts remain consistent. Mastering at least one editor is critical for system administration and development work.

## Detailed Explanation

### Core Editor Concepts

| Concept | Description |
|----------|-------------|
| **Modes** | vi/vim have insert/command modes; nano/emacs don't |
| **Buffers** | In-memory representation of file content |
| **Cut/Copy/Paste** | Delete, yank (copy), put (paste) |
| **Search/Replace** | Find text patterns, optionally replace |
| **Undo/Redo** | Revert or reapply changes |
| **Save/Exit** | Write changes to disk, close editor |

### Common Operations Across Editors

```bash
# Opening files
nano filename.txt
vim filename.txt
emacs filename.txt

# Opening at specific line
nano +10 filename.txt      # Line 10
vim +10 filename.txt
emacs +10 filename.txt

# Opening multiple files
nano file1.txt file2.txt
vim file1.txt file2.txt     # :n/:p to switch
emacs file1.txt file2.txt    # C-x b to switch

# File saving
# All: Ctrl+O (nano) :w (vim) C-x C-s (emacs)
```

### Editor Comparison

| Feature | Nano | Vim | Emacs |
|----------|-------|------|-------|
| **Learning Curve** | Easy | Hard | Very Hard |
| **Modes** | Single | Multiple | Multiple |
| **Mouse Support** | Basic | Possible | Yes |
| **Startup Time** | Fast | Fast | Slower |
| **Customization** | Limited | High (.vimrc) | Extensive (.emacs) |
| **Syntax Highlighting** | Yes | Yes | Yes |
| **Plugins** | Limited | Yes | Yes (packages) |
| **Terminal UI** | Simple | Rich | Rich |
| **GUI Version** | No | GVim | Emacs GUI |

### File Operations

```bash
# Reading files into editor
:r filename        # Vim: read file into buffer
C-x C-f           # Emacs: find file

# Writing/Saving
:w                 # Vim: write file
:saveas file.txt   # Vim: save as
C-x C-s            # Emacs: save
C-x C-w            # Emacs: write as

# Closing without saving
:q                 # Vim: quit (no changes)
:q!                # Vim: quit without saving
C-x C-c            # Emacs: quit (prompts to save)
Ctrl+X             # Nano: exit (prompts to save)
```

### Navigation

```bash
# Basic movement (arrows work in all)
j, k, h, l         # Vim: down, up, left, right (hjkl)
C-n, C-p, C-b, C-f # Emacs: next, prev, back, forward

# Word movement
w, b                # Vim: next/previous word
Esc-f, Esc-b        # Emacs: next/previous word
Alt-f, Alt-b        # Emacs: alternative next/prev word

# Line movement
0, $                # Vim: start/end of line
C-a, C-e            # Emacs: start/end of line

# File movement
gg, G               # Vim: first/last line
Esc-<, Esc->        # Emacs: first/last line
```

### Search and Replace

```bash
# Search (forward)
/pattern            # Vim: search forward
Ctrl+W              # Nano: search
C-s                 # Emacs: isearch forward

# Search (backward)
?pattern            # Vim: search backward
Ctrl+Q              # Nano: reverse search
C-r                 # Emacs: isearch backward

# Replace
:%s/old/new/g       # Vim: replace all in file
:s/old/new/gc       # Vim: replace with confirmation
Alt+%               # Emacs: query-replace
M-%                 # Emacs: query-replace

# Replace in selection
:'<,'>s/old/new/g   # Vim: in visual selection
```

### Cut, Copy, Paste

```bash
# Vim
v/V                # Enter visual mode (character/line)
d                  # Delete (cut)
y                  # Yank (copy)
p                  # Paste after cursor
P                  # Paste before cursor
dd                  # Delete line
yy                  # Yank line

# Nano
Ctrl+K             # Cut line
Ctrl+U              # Uncut (paste)
Alt+^               # Mark start, Alt+} copy

# Emacs
C-k                 # Cut (kill) line
C-y                 # Yank (paste)
C-space             # Set mark
C-w                 # Cut region
M-w                 # Copy region
C-y                 # Yank (paste)
```

### Undo and Redo

```bash
# Vim
u                   # Undo
C-r                 # Redo

# Nano
Alt+U               # Undo (limited)

# Emacs
C-/                 # Undo
C-x u               # Undo (alternative)
C-g C-/             # Redo
```

### Common Editing Patterns

```bash
# Pattern 1: Find and replace in multiple files
# Approach: Use editor with grep/sed integration
# Vim: :arg files.txt | :argdo %s/old/new/g | update

# Pattern 2: Bulk text editing
# Vim: visual block mode (Ctrl+v)
# Select column of text, edit, applies to all lines

# Pattern 3: Remote editing
vim scp://user@host:/path/to/file
emacs /ssh:user@host:/path/to/file

# Pattern 4: Split windows
:split              # Vim: horizontal split
:vsplit             # Vim: vertical split
C-x 2               # Emacs: split horizontally
C-x 3               # Emacs: split vertically
```

### Configuration Files

```bash
# Nano
~/.nanorc           # Configuration file
set const            # Display line numbers
set autoindent       # Auto indentation
set tabsize 4       # Tab width

# Vim
~/.vimrc            # Configuration
set number           # Line numbers
set autoindent       # Auto indentation
set tabstop=4        # Tab width
set expandtab       # Use spaces instead of tabs

# Emacs
~/.emacs or ~/.emacs.d/init.el
(setq column-number-mode t)
(setq-default indent-tabs-mode nil)
(setq-default tab-width 4)
```

### Choosing an Editor

```bash
# Factors to consider:
# 1. Learning curve - Nano < Vim < Emacs
# 2. Availability - All on Linux/Unix, vi always available
# 3. Customization - Vim < Emacs
# 4. Speed - Vim > Emacs (startup)
# 5. Community - Vim, Emacs large; Nano smaller

# Recommendations:
# - Nano: Beginners, quick edits, minimal configuration
# - Vim: Power users, speed, widespread availability
# - Emacs: Heavy customization, integrated environment
```

## Interview Questions

**Q: Why learn vi/vim instead of nano?**
**A:** Vim is available on virtually all Unix systems, making it universally useful. Vim's modes and powerful commands enable very fast editing once learned. Nano is simpler but lacks advanced features like macros, plugins, and extensive customization.

**Q: What is the difference between command mode and insert mode in vi/vim?**
**A:** Command mode is for navigation, deletion, copying, and other operations (default on opening). Insert mode is for typing text. Press `i`, `a`, or `o` to enter insert mode; `Esc` to return to command mode. This modal editing is unique to vi/vim.

**Q: How do you save and exit in different editors?**
**A:** Nano: `Ctrl+X` then `Y` to save. Vim: `:w` to write, `:q` to quit, `:wq` for both. Emacs: `Ctrl+X, Ctrl+S` to save, `Ctrl+X, Ctrl+C` to quit.

**Q: What is visual mode in Vim?**
**A:** Visual mode allows selecting text for operations like copy, delete, or replace. Press `v` for character selection, `V` for line selection, `Ctrl+V` for block selection. Then use commands like `y` to copy, `d` to delete, or `c` to change.

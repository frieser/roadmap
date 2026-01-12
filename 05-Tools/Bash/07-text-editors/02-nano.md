---
tags: ['nano', 'editor', 'linux', 'tools', 'roadmap']
---

# Nano

## Summary

**Nano** (GNU Nano) is a simple, user-friendly text editor for Unix/Linux systems. Named after the fictional nanotechnology in "Pulp Fiction", Nano was created as a free replacement for the non-free Pico editor. It's ideal for beginners due to its intuitive interface and minimal learning curve. Nano is included in most Linux distributions by default.

## Detailed Explanation

### Why Nano?

| Feature | Description |
|---------|-------------|
| **Easy to Learn** | No modes, intuitive keybindings |
| **Always Available** | Pre-installed on most distros |
| **Simple UI** | Clear on-screen shortcuts |
| **Minimal Config** | Works well out of box |
| **Quick Edits** | Perfect for config files |

### Nano vs Other Editors

| Editor | Learning Curve | Features | Availability |
|---------|--------------|-----------|--------------|
| **Nano** | Easy | Basic editing | Pre-installed |
| **Vim** | Hard | Extensive, powerful | Pre-installed |
| **Emacs** | Very Hard | Entire ecosystem | Pre-installed |

### Basic Usage

```bash
# Opening files
nano                    # Open new file
nano filename.txt         # Open specific file
nano +10 filename.txt    # Open at line 10
nano -r filename.txt     # Read-only mode

# Opening multiple files
nano file1.txt file2.txt file3.txt
# Use Alt+> and Alt+< to switch between files
```

### Nano Interface

```
  GNU nano 6.2                         file.txt
                                          Modified
^G Get Help  ^O Write Out  ^R Read File ^Y Prev Page ^K Cut Text
^C Cur Pos   ^X Exit      ^\ Replace   ^V Next Page ^U Uncut Text
^T Go To Ln  ^J Justify   ^^ Mark Text ^W Where Is  ^M First Lin
^Q Replace   ^B Backward  ^] To Bracket ^P PrevChar  ^V End-of-Line
^F Forward   ^M Next Line ^B PrevLine  ^L Refresh   ^E End-of-Line
```

**Legend:**
- `^` = Ctrl key
- `M-` = Alt (or Meta) key

### Essential Shortcuts

#### Navigation

| Shortcut | Action |
|----------|--------|
| `Ctrl+A` | Beginning of line |
| `Ctrl+E` | End of line |
| `Ctrl+Y` | Previous page |
| `Ctrl+V` | Next page |
| `Ctrl+F` | Forward one character |
| `Ctrl+B` | Backward one character |
| `Ctrl+Space` | Forward one word |
| `Alt+Space` | Backward one word |

#### Editing

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Cut line (stores in cut buffer) |
| `Ctrl+U` | Uncut (paste) last cut line |
| `Ctrl+6` | Mark start of selection |
| `Alt+6` | Copy selection |
| `Alt+]` | Go to matching bracket |
| `Ctrl+T` | Execute spell check (if installed) |
| `Alt+T` | Execute formatter (if installed) |

#### Search and Replace

| Shortcut | Action |
|----------|--------|
| `Ctrl+W` | Search forward |
| `Ctrl+Q` | Search backward |
| `Alt+W` | Go to line |
| `Ctrl+\` | Replace (with confirmation) |

| Search Options | Description |
|--------------|-------------|
| `^R` | Read file to insert as search string |
| `B` | Backward search |
| `C` | Case-sensitive search |
| `R` | Regular expressions |

#### File Operations

| Shortcut | Action |
|----------|--------|
| `Ctrl+O` | Write out (save) |
| `Ctrl+X` | Exit (asks to save if modified) |
| `Ctrl+R` | Read file (insert file) |
| `Ctrl+G` | Display help |

### Save and Exit

```bash
# Save without exiting
# Press: Ctrl+O
# Press: Enter (to confirm filename)
# Press: Ctrl+X (to exit) or continue editing

# Save and exit
# Press: Ctrl+X
# Press: Y (to save)
# Press: Enter (to confirm filename)

# Exit without saving
# Press: Ctrl+X
# Press: N (to discard changes)
```

### Multiple Files

```bash
# Open multiple files
nano file1.txt file2.txt file3.txt

# Switch between files
Alt+>              # Next file
Alt+<              # Previous file

# File buffer
Ctrl+^              # Go to file list
```

### Copying and Pasting

```bash
# Cut line
Ctrl+K              # Cut current line (adds to buffer)
Ctrl+K (multiple)  # Cut consecutive lines

# Paste cut lines
Ctrl+U              # Paste last cut line
Ctrl+U (multiple)  # Paste entire buffer

# Copy without cutting
Alt+6               # Copy selection (after marking)
```

### Search and Replace Workflow

```bash
# 1. Press Ctrl+\ (replace)
# 2. Enter search string (e.g., "old")
# 3. Press Enter
# 4. Enter replacement string (e.g., "new")
# 5. Press Enter
# 6. Choose option:
#    - Y: Replace this occurrence
#    - N: Skip
#    - A: Replace all (in selection)
#    - ^C: Cancel
```

### Configuration (.nanorc)

```bash
# ~/.nanorc
# Enable line numbers
set const
set linenumbers

# Auto-indentation
set autoindent
set tabsize 4

# Syntax highlighting
include "/usr/share/nano/*.nanorc"

# Backup files
set backup

# Wrap long lines
set wrap

# Mouse support
set mouse
```

### Nano Options

```bash
# Command-line options
nano -c              # Constant cursor position display
nano -m              # Enable mouse
nano -l              # Don't follow symlinks
nano -r              # Restricted mode (no file writing outside dir)
nano -v              # View (read-only) mode
nano -x              # Don't use help lines; use blank
nano +10              # Start at line 10
nano -I filename      # Indent new lines to same as previous
```

### Practical Examples

```bash
# Example 1: Edit system configuration
sudo nano /etc/hosts
# Add entry: 127.0.0.1   myapp.local
# Ctrl+O, Enter, Ctrl+X

# Example 2: Create and edit script
nano deploy.sh
# Type:
#!/bin/bash
echo "Deploying..."
# Ctrl+O, Enter, Ctrl+X
chmod +x deploy.sh

# Example 3: Quick file search
nano /var/log/syslog
# Press: Ctrl+W
# Type: error
# Press: Enter repeatedly to find next occurrences

# Example 4: Replace across file
nano config.txt
# Press: Ctrl+\
# Search: dev
# Replace: prod
# Press: A to replace all
# Ctrl+O, Ctrl+X
```

### Nano Limitations

```bash
# What Nano CAN'T do (or does poorly):
# - Syntax highlighting is basic
# - No macros or complex automation
# - Limited configuration options
# - No plugin ecosystem
# - No integrated terminal
# - No split windows
# - Limited undo functionality (Alt+U works but limited)

# For advanced features, use Vim or Emacs
```

### Nano Best Practices

```bash
# 1. Enable line numbers for debugging
# In ~/.nanorc: set linenumbers

# 2. Use tabs or spaces consistently
# In ~/.nanorc: set tabsize 4

# 3. Enable auto-indent for code
# In ~/.nanorc: set autoindent

# 4. Make backups when editing critical files
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sudo nano /etc/ssh/sshd_config

# 5. Use mouse if available (easier for beginners)
nano -m filename.txt
```

### Nano vs Vim for Beginners

```bash
# Why choose Nano:
# - No modal editing (just type)
# - Visible shortcuts at bottom
# - Faster to learn (hours vs days)
# - Good for quick edits

# Why learn Vim eventually:
# - Universally available (even in recovery mode)
# - Much faster editing once mastered
# - Powerful features (macros, plugins)
# - Industry standard for developers
```

## Interview Questions

**Q: What are the main advantages of nano over vi/vim?**
**A:** Nano has no modes (always in edit mode), displays shortcuts on screen for reference, and has a gentle learning curve. You can simply open and start typing. Vi/vim requires learning modes and commands before being productive.

**Q: How do you save and exit in nano?**
**A:** Press `Ctrl+O` to write (save), then `Enter` to confirm filename. Press `Ctrl+X` to exit. If file was modified, nano asks if you want to save (Y for yes, N for no).

**Q: How do you perform search and replace in nano?**
**A:** Press `Ctrl+\` to enter replace mode. Enter search string, press `Enter`, enter replacement string, press `Enter`. Then press `Y` to replace each occurrence, `A` to replace all, `N` to skip, or `Ctrl+C` to cancel.

**Q: When should you use nano instead of vim?**
**A:** Use nano for quick config file edits, when you don't use the editor often, or when teaching someone new to command line. Use vim for development, complex editing tasks, or when you need advanced features like macros and plugins.

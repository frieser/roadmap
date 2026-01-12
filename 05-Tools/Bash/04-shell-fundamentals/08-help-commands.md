---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Help Commands

## Summary

Bash provides multiple ways to get help for commands: `man` (manual pages), `help` (shell builtins), `info` (GNU documentation), `--help` flags, and `whatis`/`apropos` for discovery. Knowing how to find documentation quickly is essential for efficient shell usage and learning new commands.

## Detailed Explanation

### Manual Pages (man)

```bash
# Read the manual for a command
man ls
man grep
man bash

# Manual sections
man 1 passwd      # User commands
man 5 passwd      # File formats (/etc/passwd)

# Section numbers:
# 1 - User commands
# 2 - System calls
# 3 - Library functions
# 4 - Device files
# 5 - File formats
# 6 - Games
# 7 - Miscellaneous
# 8 - System administration

# Navigate man pages
# Space/f  - next page
# b        - previous page
# /pattern - search forward
# n        - next match
# q        - quit
```

### Shell Builtin Help

```bash
# help - for Bash builtins only
help cd
help for
help if

# List all builtins
help

# Short description
help -d cd

# Check if command is builtin
type cd          # cd is a shell builtin
type ls          # ls is /bin/ls
```

### --help Flag

```bash
# Most commands support --help
ls --help
grep --help
docker --help

# Short version (sometimes)
ls -h             # May conflict with other options
command -?        # Some commands use this
```

### Info Pages (GNU)

```bash
# Detailed GNU documentation
info bash
info coreutils

# Navigation
# Enter   - follow link
# u       - up one level
# n/p     - next/previous section
# q       - quit

# Many prefer man, but info has more detail for GNU tools
```

### Discovery Commands

```bash
# whatis - one-line description
whatis grep
# grep (1) - print lines that match patterns

# apropos - search manual descriptions
apropos search
apropos "copy files"

# Equivalent to: man -k
man -k password

# which - find executable location
which python
# /usr/bin/python

# whereis - find binary, source, and man page
whereis ls
# ls: /bin/ls /usr/share/man/man1/ls.1.gz

# type - show how command would be interpreted
type ls          # ls is aliased to 'ls --color=auto'
type -a ls       # Show all matches
```

### Command Information

```bash
# file - determine file type
file /bin/ls
# /bin/ls: ELF 64-bit LSB pie executable...

# tldr - simplified man pages (community project)
# Install: npm install -g tldr
tldr tar

# Example output:
# tar
# Archiving utility
# 
# - Create archive:
#   tar cf archive.tar file1 file2
# 
# - Extract archive:
#   tar xf archive.tar
```

### Quick Reference

| Need | Command |
|------|---------|
| Full documentation | `man command` |
| Bash builtin help | `help builtin` |
| Quick usage | `command --help` |
| What is it? | `whatis command` |
| Find by keyword | `apropos keyword` |
| Command location | `which command` |
| Command type | `type command` |

## Interview Questions

**Q: What is the difference between `man` and `help`?**
**A:** `man` shows manual pages for external commands and system calls. `help` shows help for Bash shell builtins like `cd`, `echo`, `for`, `if`. Use `help` for builtins, `man` for external programs.

**Q: How do you find commands related to a keyword?**
**A:** Use `apropos keyword` or `man -k keyword`. This searches the one-line descriptions of all manual pages. For example, `apropos compress` finds commands related to compression.

**Q: What is the difference between `which` and `type`?**
**A:** `which` finds the executable in PATH. `type` shows how Bash would interpret the command - it reveals aliases, functions, builtins, and executables. `type` is more informative for shell usage.

**Q: How do you navigate inside man pages?**
**A:** Use Space/f to page forward, b to page back, `/pattern` to search, n for next match, q to quit. Man pages use the `less` pager by default, so all less commands work.

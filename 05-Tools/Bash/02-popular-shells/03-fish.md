---
tags: ['fish', 'shell', 'linux', 'tools', 'roadmap']
---

# Fish (Friendly Interactive Shell)

## Summary

**Fish** (Friendly Interactive Shell) is a modern, user-friendly shell designed for interactive use. Created in 2005, it prioritizes discoverability and ease of use over POSIX compatibility. Fish features syntax highlighting, autosuggestions based on history, and web-based configuration. It's ideal for users who want a powerful shell that works well out of the box.

## Detailed Explanation

### Key Features

| Feature | Description |
|---------|-------------|
| **Syntax Highlighting** | Colors indicate valid/invalid commands as you type |
| **Autosuggestions** | Ghost text from history (accept with →) |
| **Web Config** | `fish_config` opens browser-based settings |
| **Universal Variables** | Variables shared across all sessions instantly |
| **No Configuration Needed** | Sensible defaults out of the box |
| **Tab Completion** | Rich completion with descriptions |

### Fish Syntax Differences

```fish
# Variables (no $ for assignment)
set name "World"
echo "Hello, $name"

# Functions (not POSIX compatible)
function greet
    echo "Hello, $argv[1]"
end

# Conditionals (no [[ or [ )
if test -f ~/.config/fish/config.fish
    echo "Config exists"
end

# Loops
for file in *.txt
    echo $file
end

# No subshell $(...)
set result (command)    # Use ( ) instead

# Lists/arrays
set mylist a b c
echo $mylist[1]         # "a" (1-indexed)
```

### Configuration

```fish
# Config file location
~/.config/fish/config.fish

# Set environment variable
set -x PATH $HOME/bin $PATH

# Universal variable (persists everywhere)
set -U EDITOR vim

# Add to path (fish-specific)
fish_add_path ~/.local/bin
```

### Autosuggestions in Action

```fish
# Type partial command, see ghost text from history
$ git com<gray>mit -m "fix typo"</gray>
# Press → to accept, or keep typing

# Abbreviations (expand as you type)
abbr -a gc 'git commit'
# Type "gc " and it expands to "git commit "
```

### Interactive Features

```fish
# Open web config
fish_config

# Change prompt theme
fish_config prompt

# Private mode (no history)
fish --private
```

## Interview Questions

**Q: Why doesn't Fish follow POSIX compatibility?**
**A:** Fish prioritizes user experience and modern features over backward compatibility. POSIX syntax has legacy constraints that conflict with Fish's goals of discoverability and sensible defaults. Scripts for Fish stay in Fish; portable scripts use Bash or sh.

**Q: What is the main advantage of Fish for new users?**
**A:** It works great out of the box - syntax highlighting, autosuggestions, and tab completion require no configuration. Users can be productive immediately without learning complex configuration.

**Q: When should you NOT use Fish?**
**A:** For writing portable scripts that run on any system. Fish scripts only work in Fish. Use `#!/bin/bash` or `#!/bin/sh` for portable scripts. Fish is best for interactive use.

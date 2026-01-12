---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Setting Up Bash

## Summary

Setting up Bash involves configuring initialization files, customizing the prompt, setting environment variables, and creating aliases and functions. The key files are `~/.bashrc` for interactive shells and `~/.bash_profile` for login shells. Proper setup improves productivity through customized prompts, useful aliases, and consistent environment across sessions.

## Detailed Explanation

### Configuration File Hierarchy

```mermaid
flowchart TD
    A[Shell Start] --> B{Login Shell?}
    B -->|Yes| C[/etc/profile]
    C --> D[~/.bash_profile]
    D --> E{~/.bash_login?}
    E -->|No| F[~/.profile]
    B -->|No| G[~/.bashrc]
    D -.-> G
    F -.-> G
```

### Essential Configuration Files

```bash
# ~/.bash_profile - Login shells (SSH, console login)
# This file should source .bashrc for consistent behavior

# Check for interactive shell
if [ -n "$PS1" ]; then
    # Source .bashrc if it exists
    if [ -f ~/.bashrc ]; then
        source ~/.bashrc
    fi
fi

# Login-specific settings
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"
```

```bash
# ~/.bashrc - Interactive non-login shells
# Most customization goes here

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# History settings
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoreboth:erasedups
shopt -s histappend

# Check window size after each command
shopt -s checkwinsize

# Enable programmable completion
if [ -f /etc/bash_completion ]; then
    source /etc/bash_completion
fi
```

### Customizing the Prompt (PS1)

```bash
# Basic prompt
PS1='\u@\h:\w\$ '
# Result: user@hostname:/current/path$

# Colored prompt
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# Prompt escape sequences
# \u - Username
# \h - Hostname (short)
# \H - Hostname (full)
# \w - Current directory (full path)
# \W - Current directory (basename only)
# \$ - $ for user, # for root
# \t - Time (24-hour)
# \d - Date
# \n - Newline

# Multi-line prompt with git branch
parse_git_branch() {
    git branch 2>/dev/null | sed -n 's/* \(.*\)/ (\1)/p'
}
PS1='\[\033[32m\]\u@\h\[\033[00m\]:\[\033[34m\]\w\[\033[33m\]$(parse_git_branch)\[\033[00m\]\n\$ '
```

### Useful Aliases

```bash
# Add to ~/.bashrc

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ~='cd ~'

# Listing
alias ls='ls --color=auto'
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Safety nets
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Grep colors
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'

# System
alias df='df -h'
alias du='du -h'
alias free='free -m'

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline'
```

### Environment Variables

```bash
# Add to ~/.bashrc or ~/.bash_profile

# Default editor
export EDITOR=vim
export VISUAL=vim

# Path additions
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# Language/Locale
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

# Application-specific
export GOPATH="$HOME/go"
export PATH="$PATH:$GOPATH/bin"
```

### Shell Options (shopt)

```bash
# Useful shell options
shopt -s cdspell        # Autocorrect minor typos in cd
shopt -s dirspell       # Autocorrect directory names
shopt -s autocd         # Type directory name to cd into it
shopt -s globstar       # ** matches recursively
shopt -s nocaseglob     # Case-insensitive globbing
shopt -s histappend     # Append to history, don't overwrite
shopt -s cmdhist        # Save multi-line commands as one
```

### Applying Changes

```bash
# Reload configuration after editing
source ~/.bashrc
# or
. ~/.bashrc

# Start new shell to test
bash

# Check current configuration
echo $PS1
alias
```

## Interview Questions

**Q: What is the difference between ~/.bashrc and ~/.bash_profile?**
**A:** `~/.bash_profile` runs for login shells (console login, SSH). `~/.bashrc` runs for interactive non-login shells (new terminal window). Best practice: put all config in `.bashrc` and source it from `.bash_profile`.

**Q: How do you make an alias permanent?**
**A:** Add the alias command to `~/.bashrc`, then run `source ~/.bashrc` or open a new terminal. Aliases defined in the terminal are session-only and lost when the shell closes.

**Q: What does `export` do when setting environment variables?**
**A:** `export` makes the variable available to child processes. Without export, the variable is only available in the current shell. For example, `PATH=/new/path` only affects the current shell; `export PATH` makes it available to programs you run.

**Q: How do you add a directory to PATH permanently?**
**A:** Add `export PATH="$HOME/bin:$PATH"` to `~/.bashrc`. The order matters - directories listed first take precedence. Prepend for priority, append if fallback is acceptable.

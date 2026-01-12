---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Bash Aliases

## Summary

**Aliases** are shortcuts for frequently used commands. They replace a word with a string when used as the first word of a simple command. Aliases save typing, reduce errors, and customize your shell environment. They are defined in `~/.bashrc` for persistence across sessions.

## Detailed Explanation

### Creating Aliases

```bash
# Basic syntax
alias name='command'

# Simple shortcuts
alias ll='ls -la'
alias la='ls -A'
alias l='ls -CF'

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias ~='cd ~'

# Safety nets
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Colorize output
alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias diff='diff --color=auto'
```

### Viewing and Removing Aliases

```bash
# List all aliases
alias

# Show specific alias
alias ll

# Remove an alias
unalias ll

# Remove all aliases
unalias -a

# Bypass alias (run original command)
\ls                # Backslash
'ls'               # Quotes
command ls         # command builtin
/bin/ls            # Full path
```

### Making Aliases Permanent

```bash
# Add to ~/.bashrc
cat >> ~/.bashrc << 'EOF'

# Custom aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias gs='git status'
alias gc='git commit'
alias gp='git push'

EOF

# Apply changes
source ~/.bashrc
```

### Alias Limitations

```bash
# Aliases don't work in scripts (by default)
# Enable with: shopt -s expand_aliases

# Aliases can't take arguments directly
alias greet='echo Hello'
greet World        # Outputs: Hello World (not what you might expect)
                   # Actually: echo Hello World

# For arguments, use functions instead
greet() {
    echo "Hello, $1!"
}
greet World        # Outputs: Hello, World!

# Aliases are simple substitution
alias list='ls'
list -la           # Works: becomes ls -la

alias la='ls -la'  
la -h              # Works: becomes ls -la -h
```

### Useful Alias Collections

```bash
# System administration
alias df='df -h'
alias du='du -h'
alias free='free -m'
alias psg='ps aux | grep -v grep | grep'

# Git aliases
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph'
alias gd='git diff'
alias gb='git branch'
alias gco='git checkout'

# Docker aliases
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias dex='docker exec -it'

# Directory shortcuts
alias proj='cd ~/projects'
alias docs='cd ~/Documents'
alias dl='cd ~/Downloads'

# Quick edits
alias bashrc='${EDITOR:-vim} ~/.bashrc'
alias reload='source ~/.bashrc'
```

### Alias vs Function

```bash
# Use alias for: simple command substitution
alias ll='ls -la'

# Use function for: arguments, logic, multiple commands
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Function can handle complex scenarios
extract() {
    case "$1" in
        *.tar.gz)  tar xzf "$1" ;;
        *.tar.bz2) tar xjf "$1" ;;
        *.zip)     unzip "$1" ;;
        *)         echo "Unknown format" ;;
    esac
}
```

## Interview Questions

**Q: What is the difference between an alias and a function?**
**A:** Aliases are simple text substitutions; they can't handle arguments flexibly or contain logic. Functions can take parameters (`$1`, `$2`), contain conditionals and loops, and execute multiple commands with complex logic.

**Q: How do you run the original command when an alias exists?**
**A:** Prefix with backslash (`\rm`), use quotes (`'rm'`), use the `command` builtin (`command rm`), or use the full path (`/bin/rm`).

**Q: Why don't aliases work in shell scripts by default?**
**A:** Bash disables alias expansion in non-interactive mode for predictability. Scripts should be explicit. To enable, use `shopt -s expand_aliases`, but prefer functions for scripts.

**Q: Where should you define aliases for them to persist?**
**A:** In `~/.bashrc` for interactive shells. If you use a separate file like `~/.bash_aliases`, source it from `.bashrc`: `source ~/.bash_aliases`.

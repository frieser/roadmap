---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Tab Completion

## Summary

**Tab completion** is a productivity feature that completes commands, filenames, and arguments when you press the Tab key. Bash provides programmable completion that can be customized for specific commands. Mastering tab completion significantly speeds up command-line work and reduces typing errors.

## Detailed Explanation

### Basic Tab Completion

```bash
# Complete file/directory names
cd Doc<TAB>          # → cd Documents/

# Multiple matches: show options
cd D<TAB><TAB>       # Shows: Desktop/ Documents/ Downloads/

# Complete commands
sys<TAB><TAB>        # Shows: systemctl, systemd, etc.

# Complete from any position
cat /etc/pass<TAB>   # → cat /etc/passwd
```

### Readline Shortcuts

```bash
# Tab completion
TAB          # Complete word
TAB TAB      # Show all matches

# Escape completion
ESC ?        # Show possible completions
ESC *        # Insert all completions

# Case-insensitive completion (add to ~/.inputrc)
set completion-ignore-case on

# Show all if ambiguous (add to ~/.inputrc)
set show-all-if-ambiguous on
```

### Configuration in ~/.inputrc

```bash
# Create or edit ~/.inputrc
cat >> ~/.inputrc << 'EOF'
# Case-insensitive completion
set completion-ignore-case on

# Show all matches on first tab
set show-all-if-ambiguous on

# Show file type with colors
set colored-stats on

# Append slash to directories
set mark-directories on

# Show common prefix immediately
set completion-prefix-display-length 2

# Don't expand ~ to home directory
set expand-tilde off
EOF

# Apply changes
bind -f ~/.inputrc
```

### Programmable Completion

```bash
# See current completions
complete -p

# See completion for specific command
complete -p git

# Basic custom completion
complete -W "start stop restart status" myservice

# Test it
myservice st<TAB><TAB>    # Shows: start status stop

# Complete with files of specific type
complete -f -X '!*.txt' mycommand
# Only .txt files complete for mycommand
```

### Common Built-in Completions

```bash
# Enable bash-completion package (usually pre-installed)
source /etc/bash_completion
# or
source /usr/share/bash-completion/bash_completion

# Now you get smart completion for:
git che<TAB>         # → git checkout
ssh user@<TAB>       # Completes hostnames from known_hosts
scp file.txt <TAB>   # Completes remote paths
apt install ng<TAB>  # Completes package names
```

### Creating Custom Completion

```bash
# Add to ~/.bashrc

# Simple word list completion
_myapp_completions() {
    local cur=${COMP_WORDS[COMP_CWORD]}
    COMPREPLY=($(compgen -W "start stop restart configure" -- "$cur"))
}
complete -F _myapp_completions myapp

# Context-aware completion
_myapp_completions() {
    local cur prev
    cur=${COMP_WORDS[COMP_CWORD]}
    prev=${COMP_WORDS[COMP_CWORD-1]}
    
    case "$prev" in
        configure)
            COMPREPLY=($(compgen -W "port timeout debug" -- "$cur"))
            ;;
        *)
            COMPREPLY=($(compgen -W "start stop configure" -- "$cur"))
            ;;
    esac
}
complete -F _myapp_completions myapp
```

### Useful Completion Tips

```bash
# Complete environment variable names
echo $PA<TAB>        # Shows $PATH, $PAGER, etc.

# Complete user names
chown <TAB><TAB>     # Shows usernames

# Complete hostnames (from /etc/hosts and known_hosts)
ssh <TAB><TAB>

# Complete from history
Ctrl+R               # Reverse search history

# Cycle through completions (add to ~/.inputrc)
TAB: menu-complete
```

## Interview Questions

**Q: How do you enable case-insensitive tab completion?**
**A:** Add `set completion-ignore-case on` to `~/.inputrc` and restart your shell or run `bind -f ~/.inputrc`. This makes `cd doc<TAB>` complete to `Documents/`.

**Q: What is programmable completion?**
**A:** Bash's ability to provide context-aware completions for specific commands. For example, `git checkout <TAB>` shows branches, not files. It's configured with the `complete` builtin and completion functions.

**Q: How do you see all possible completions immediately?**
**A:** Press Tab twice, or add `set show-all-if-ambiguous on` to `~/.inputrc` to show all matches on the first Tab press when there are multiple options.

**Q: Where are completion scripts typically stored?**
**A:** System-wide in `/etc/bash_completion.d/` or `/usr/share/bash-completion/completions/`. User scripts can go in `~/.local/share/bash-completion/completions/` or be sourced from `~/.bashrc`.

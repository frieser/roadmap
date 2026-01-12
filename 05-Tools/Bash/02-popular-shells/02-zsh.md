---
tags: ['zsh', 'shell', 'linux', 'tools', 'roadmap']
---

# Zsh (Z Shell)

## Summary

**Zsh** (Z Shell) is an extended Bourne shell with many improvements, including better tab completion, spelling correction, and extensive customization. Created by Paul Falstad in 1990, it became the default shell on macOS starting with Catalina (10.15). Zsh combines features from bash, ksh, and tcsh, and is highly popular due to frameworks like Oh My Zsh.

## Detailed Explanation

### Key Features

| Feature | Description |
|---------|-------------|
| **Smart Completion** | Context-aware, programmable tab completion |
| **Globbing** | Extended pattern matching (`**/*.txt`) |
| **Spelling Correction** | Auto-correct typos |
| **Themes/Prompts** | Powerful prompt customization |
| **Plugin System** | Extensible via frameworks (Oh My Zsh, Prezto) |
| **Shared History** | History shared across all sessions |
| **Floating Point** | Native arithmetic with decimals |

### Zsh vs Bash

```bash
# Recursive globbing (works in Zsh natively, needs shopt in Bash)
ls **/*.md

# Array indexing (Zsh is 1-based, Bash is 0-based)
arr=(a b c)
echo ${arr[1]}    # Zsh: "a", Bash: "b"

# Spelling correction
$ cd /usrr/locale
zsh: correct '/usrr/locale' to '/usr/locale' [nyae]?

# Suffix aliases
alias -s md=vim   # Typing file.md opens it in vim

# Global aliases
alias -g G='| grep'
ls G pattern      # Expands to: ls | grep pattern
```

### Configuration Files

```bash
# Zsh configuration (in order):
~/.zshenv          # Always loaded (environment vars)
~/.zprofile        # Login shells only
~/.zshrc           # Interactive shells (most customization)
~/.zlogin          # Login shells, after .zshrc
~/.zlogout         # When exiting login shell

# Most configuration goes in ~/.zshrc
```

### Oh My Zsh Setup

```bash
# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Configure in ~/.zshrc
ZSH_THEME="robbyrussell"
plugins=(git docker kubectl aws)

source $ZSH/oh-my-zsh.sh
```

### Zsh-Specific Features

```bash
# Named directories
hash -d proj=~/projects
cd ~proj          # Expands to ~/projects

# Zmv - batch rename
autoload zmv
zmv '(*).txt' '$1.md'    # Rename all .txt to .md

# Right-side prompt
RPROMPT='%T'      # Shows time on right

# Associative arrays (like Bash 4+)
typeset -A map
map[key]=value
```

## Interview Questions

**Q: Why has Zsh become so popular among developers?**
**A:** Superior tab completion with descriptions, extensive customization through frameworks like Oh My Zsh, powerful globbing, spelling correction, and rich plugin ecosystem. Its interactive experience is more polished than Bash.

**Q: What is the main compatibility concern between Zsh and Bash?**
**A:** Array indexing: Zsh is 1-based while Bash is 0-based. Also, some Bash-specific syntax like `[[` behaves slightly differently. Scripts meant for Bash should use `#!/bin/bash` to ensure compatibility.

**Q: When would you choose Zsh over Bash?**
**A:** For interactive daily use - better completion, themes, plugins make it more productive. For scripts intended to run on many systems, Bash or POSIX sh is safer due to wider availability.

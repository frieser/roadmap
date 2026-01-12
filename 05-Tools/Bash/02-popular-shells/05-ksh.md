---
tags: ['ksh', 'shell', 'linux', 'tools', 'roadmap']
---

# Ksh (KornShell)

## Summary

**KornShell** (ksh), developed by David Korn at Bell Labs in 1983, was one of the most influential Unix shells. It introduced many features later adopted by Bash, including command-line editing, job control, and associative arrays. Ksh is still the default shell on some commercial Unix systems (AIX, HP-UX, Solaris) and is valued for its speed and POSIX compliance with extensions.

## Detailed Explanation

### Ksh Versions

| Version | Description |
|---------|-------------|
| **ksh88** | Original 1988 version, still on legacy systems |
| **ksh93** | 1993 rewrite with many enhancements |
| **pdksh** | Public Domain Korn Shell (open source clone) |
| **mksh** | MirBSD Korn Shell (modern, Android default) |
| **ksh2020** | Latest AT&T ksh release |

### Features Ksh Introduced

```bash
# Command-line editing (before Bash had it)
# ESC for vi mode, or set -o emacs

# Aliases
alias ll='ls -la'

# Functions
function greet {
    print "Hello, $1"
}

# Coprocesses (two-way communication)
cmd |&
print -p "input"
read -p output

# Associative arrays (before Bash 4)
typeset -A map
map[key]="value"

# Floating-point arithmetic
print $((3.14 * 2))
# 6.28
```

### Ksh vs Bash Syntax

```bash
# Ksh uses 'print' (Bash uses 'echo')
print "Hello"           # Ksh
echo "Hello"            # Bash (also works in ksh)

# Function definition
function name {         # Ksh style
    ...
}
name() {                # POSIX style (both support)
    ...
}

# Typeset vs declare
typeset -i num          # Ksh
declare -i num          # Bash (also works in ksh)

# Reading from coprocess
cmd |&
read -p var             # Ksh
# Bash requires coproc keyword (4.0+)
```

### Configuration

```bash
# Ksh configuration files
~/.profile           # Login shells
$ENV                 # Sourced for interactive shells

# Set ENV in .profile
export ENV=~/.kshrc
```

### When You'll Encounter Ksh

```bash
# Check default shell on commercial Unix
echo $0              # Might be /bin/ksh

# Solaris, AIX, HP-UX often default to ksh
cat /etc/shells      # List available shells

# Convert ksh script to bash
# Most ksh88 scripts work in bash
# ksh93 extensions may need adjustment
```

## Interview Questions

**Q: What is the historical significance of KornShell?**
**A:** Ksh pioneered features now standard in modern shells: command-line editing, job control, aliases, functions, and associative arrays. Both Bash and Zsh adopted many ksh concepts. It influenced POSIX shell standards.

**Q: When would you encounter Ksh today?**
**A:** On commercial Unix systems (AIX, HP-UX, Solaris) where it's often the default. Legacy enterprise scripts written in ksh. Android uses mksh (MirBSD Korn Shell) as its default shell.

**Q: How compatible is Ksh with Bash?**
**A:** Highly compatible for basic scripts. Both follow POSIX with extensions. Main differences: ksh uses `print` vs `echo`, ksh88 lacks some Bash 4+ features. Most ksh scripts run in Bash; complex ksh93 features may need adjustment.

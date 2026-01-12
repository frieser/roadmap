---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Bash (Bourne Again Shell)

## Summary

**Bash** is the most widely used Unix shell, serving as the default on most Linux distributions. Created by Brian Fox in 1989 for the GNU Project, it extends the Bourne shell (sh) with features from ksh and csh. Bash provides command-line editing, job control, shell functions, arrays, and extensive scripting capabilities. Its ubiquity makes it the essential shell for system administration and automation.

## Detailed Explanation

### Key Features

| Feature | Description |
|---------|-------------|
| **Readline** | Command-line editing with vi/emacs modes |
| **History** | Command history with search (`Ctrl+R`) |
| **Tab Completion** | Programmable completion for commands and files |
| **Job Control** | Background processes, `fg`, `bg`, `jobs` |
| **Arrays** | Indexed and associative arrays (4.0+) |
| **Functions** | Shell functions with local variables |
| **Brace Expansion** | `{a,b,c}`, `{1..10}` |

### Bash Configuration Files

```bash
# Login shells (in order):
/etc/profile           # System-wide
~/.bash_profile        # User (preferred)
~/.bash_login          # User (fallback)
~/.profile             # User (fallback)

# Interactive non-login shells:
~/.bashrc              # User configuration

# Best practice: source bashrc from bash_profile
# ~/.bash_profile:
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

### Bash-Specific Syntax

```bash
# Extended test brackets
[[ $var == pattern* ]]    # Pattern matching
[[ $var =~ regex ]]       # Regex matching

# Brace expansion
echo file{1..5}.txt       # file1.txt file2.txt ...
mkdir -p project/{src,bin,lib}

# Process substitution
diff <(sort file1) <(sort file2)

# Arrays
arr=("one" "two" "three")
echo "${arr[@]}"          # All elements
echo "${#arr[@]}"         # Array length

# Associative arrays (Bash 4+)
declare -A map
map[key]="value"
```

### Checking Bash Version

```bash
bash --version
# GNU bash, version 5.1.16(1)-release

echo $BASH_VERSION
# 5.1.16(1)-release

# Version check in scripts
if ((BASH_VERSINFO[0] >= 4)); then
    declare -A assoc_array
fi
```

## Interview Questions

**Q: What makes Bash different from the original Bourne shell?**
**A:** Bash adds arrays, `[[` extended tests, brace expansion, process substitution, command-line editing, programmable completion, and many built-in commands. It's a superset maintaining backward compatibility with sh.

**Q: Where is Bash configuration stored?**
**A:** System-wide in `/etc/profile` and `/etc/bash.bashrc`. User settings in `~/.bash_profile` (login shells) and `~/.bashrc` (interactive shells). The common practice is to source `.bashrc` from `.bash_profile`.

**Q: How do you check if Bash version supports a feature?**
**A:** Check `$BASH_VERSINFO` array: `if ((BASH_VERSINFO[0] >= 4)); then ...`. For associative arrays you need Bash 4.0+; for `mapfile` you need 4.0+.

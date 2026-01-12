---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# If Statements

## Summary

Bash if statements execute code conditionally based on the exit status of commands. They support `if`, `elif`, `else`, and use test expressions with `[ ]` or `[[ ]]`. Understanding the difference between single and double brackets, and proper quoting, is essential for robust scripts.

## Detailed Explanation

### Basic Syntax

```bash
# Basic if
if [[ condition ]]; then
    commands
fi

# if-else
if [[ condition ]]; then
    commands
else
    other_commands
fi

# if-elif-else
if [[ condition1 ]]; then
    commands1
elif [[ condition2 ]]; then
    commands2
else
    default_commands
fi
```

### [ ] vs [[ ]]

```bash
# [ ] - POSIX test command
if [ "$var" = "value" ]; then ...

# [[ ]] - Bash extended test (preferred)
if [[ $var == "value" ]]; then ...

# Key differences:
# [[ ]] doesn't require quoting variables
# [[ ]] supports pattern matching (==) and regex (=~)
# [[ ]] handles && and || internally
# [ ] is more portable to other shells
```

### String Comparisons

```bash
# String equality
if [[ "$str" == "hello" ]]; then
    echo "Match"
fi

# String inequality
if [[ "$str" != "hello" ]]; then
    echo "No match"
fi

# String empty/non-empty
if [[ -z "$str" ]]; then echo "Empty"; fi
if [[ -n "$str" ]]; then echo "Not empty"; fi

# Pattern matching (globs)
if [[ "$file" == *.txt ]]; then
    echo "Text file"
fi

# Regex matching
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email"
fi
```

### Numeric Comparisons

```bash
# Use -eq, -ne, -lt, -le, -gt, -ge
if [[ $num -eq 10 ]]; then echo "Equals 10"; fi
if [[ $num -ne 10 ]]; then echo "Not 10"; fi
if [[ $num -lt 10 ]]; then echo "Less than 10"; fi
if [[ $num -le 10 ]]; then echo "Less or equal"; fi
if [[ $num -gt 10 ]]; then echo "Greater than 10"; fi
if [[ $num -ge 10 ]]; then echo "Greater or equal"; fi

# Arithmetic context (alternative)
if (( num > 10 )); then
    echo "Greater than 10"
fi

if (( num >= 5 && num <= 15 )); then
    echo "Between 5 and 15"
fi
```

### File Tests

```bash
if [[ -e "$file" ]]; then echo "Exists"; fi
if [[ -f "$file" ]]; then echo "Regular file"; fi
if [[ -d "$path" ]]; then echo "Directory"; fi
if [[ -r "$file" ]]; then echo "Readable"; fi
if [[ -w "$file" ]]; then echo "Writable"; fi
if [[ -x "$file" ]]; then echo "Executable"; fi
if [[ -s "$file" ]]; then echo "Non-empty"; fi
if [[ -L "$file" ]]; then echo "Symlink"; fi
if [[ "$f1" -nt "$f2" ]]; then echo "f1 newer"; fi
if [[ "$f1" -ot "$f2" ]]; then echo "f1 older"; fi
```

### Compound Conditions

```bash
# AND
if [[ -f "$file" && -r "$file" ]]; then
    cat "$file"
fi

# OR
if [[ -z "$var" || "$var" == "default" ]]; then
    var="fallback"
fi

# NOT
if [[ ! -e "$file" ]]; then
    echo "File doesn't exist"
fi

# Grouping
if [[ ( "$a" == "x" || "$a" == "y" ) && -n "$b" ]]; then
    echo "Complex condition"
fi
```

### Command Exit Status

```bash
# if directly on command
if grep -q "pattern" file.txt; then
    echo "Found"
fi

if ping -c 1 google.com &>/dev/null; then
    echo "Network OK"
fi

# Using command directly
if command -v docker &>/dev/null; then
    echo "Docker installed"
fi
```

### One-Liners

```bash
# && for success, || for failure
[[ -f "$file" ]] && cat "$file"
[[ -d "$dir" ]] || mkdir -p "$dir"

# Ternary-like
[[ $x -gt 5 ]] && echo "big" || echo "small"
```

## Interview Questions

**Q: What is the difference between `[ ]` and `[[ ]]`?**
**A:** `[ ]` is POSIX-compliant test command; `[[ ]]` is Bash-specific with extra features: no need to quote variables, pattern matching with `==`, regex with `=~`, and `&&`/`||` inside. Use `[[ ]]` in Bash scripts.

**Q: How do you compare numbers in Bash?**
**A:** Use `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge` inside `[[ ]]`, or use arithmetic context `(( ))` with `==`, `!=`, `<`, `<=`, `>`, `>=`.

**Q: How do you check if a command exists?**
**A:** Use `if command -v cmdname &>/dev/null; then ...`. Or `type cmdname &>/dev/null`. Avoid `which` as it's not always reliable.

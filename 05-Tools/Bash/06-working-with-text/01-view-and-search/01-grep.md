---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# grep (Global Regular Expression Print)

## Summary

`grep` searches for patterns in text files and outputs matching lines. It supports basic, extended, and Perl-compatible regular expressions. Grep is one of the most essential Unix tools for filtering text, searching logs, and finding content in files.

## Detailed Explanation

### Basic Usage

```bash
# Search for pattern in file
grep "error" logfile.txt

# Search in multiple files
grep "pattern" file1.txt file2.txt

# Search recursively
grep -r "TODO" ./src
```

### Common Options

```bash
# -i: Case insensitive
grep -i "error" log.txt

# -v: Invert match (lines NOT matching)
grep -v "debug" log.txt

# -n: Show line numbers
grep -n "function" script.py

# -c: Count matches
grep -c "error" log.txt

# -l: List files with matches
grep -rl "TODO" ./src

# -L: List files WITHOUT matches
grep -rL "license" ./

# -w: Match whole words only
grep -w "log" file.txt   # Matches "log" not "logging"

# -x: Match whole lines only
grep -x "exact line"
```

### Regular Expressions

```bash
# Basic regex (default)
grep "error.*timeout" log.txt

# Extended regex (-E or egrep)
grep -E "(error|warning)" log.txt
grep -E "[0-9]{3}-[0-9]{4}" # Phone pattern

# Perl regex (-P)
grep -P "\d{3}-\d{4}" file.txt

# Common patterns
grep "^start"         # Lines starting with "start"
grep "end$"           # Lines ending with "end"
grep "^$"             # Empty lines
grep -E "[0-9]+"      # Lines with numbers
```

### Context Options

```bash
# -A N: Show N lines after match
grep -A 3 "error" log.txt

# -B N: Show N lines before match
grep -B 2 "error" log.txt

# -C N: Show N lines before AND after
grep -C 2 "error" log.txt
```

### Practical Examples

```bash
# Find errors in logs
grep -i "error\|fail" /var/log/*.log

# Find function definitions
grep -n "^function\|^def " *.sh *.py

# Find TODOs with context
grep -rn --include="*.go" "TODO" .

# Exclude directories
grep -r --exclude-dir={.git,node_modules} "pattern" .

# Count errors per file
grep -c "ERROR" *.log | grep -v ":0$"

# Extract IPs from log
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" access.log
```

### Performance Options

```bash
# --include/--exclude file patterns
grep -r --include="*.py" "import" .

# -F: Fixed strings (faster, no regex)
grep -F "exact [string]" file.txt

# -m N: Stop after N matches
grep -m 1 "first match" file.txt
```

## Interview Questions

**Q: What is the difference between grep, egrep, and fgrep?**
**A:** `grep` uses basic regex. `egrep` (or `grep -E`) uses extended regex (supports `+`, `?`, `|`, `()` without escaping). `fgrep` (or `grep -F`) treats pattern as fixed string, no regex, faster for literal matching.

**Q: How do you search recursively while excluding certain directories?**
**A:** Use `grep -r --exclude-dir=dirname "pattern" .` For multiple: `--exclude-dir={.git,node_modules,vendor}`.

**Q: How do you extract only the matching part, not the whole line?**
**A:** Use `grep -o`. For example: `grep -oE "[0-9]+" file.txt` outputs just the numbers, one per line.

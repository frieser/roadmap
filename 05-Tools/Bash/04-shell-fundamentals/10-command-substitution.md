---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Command Substitution

## Summary

**Command substitution** allows you to capture the output of a command and use it as part of another command or assign it to a variable. The syntax is `$(command)` or backticks `` `command` ``. This is fundamental for dynamic scripting, capturing results, and building commands programmatically.

## Detailed Explanation

### Basic Syntax

```bash
# Modern syntax (preferred)
result=$(command)

# Legacy syntax (backticks)
result=`command`

# Use in commands
echo "Today is $(date)"
echo "You are $(whoami)"
echo "Current directory: $(pwd)"
```

### Why Prefer $() Over Backticks

```bash
# $() is easier to read
files=$(ls -la)

# $() nests cleanly
outer=$(echo $(cat file.txt))

# Backticks require escaping for nesting
outer=`echo \`cat file.txt\``

# $() works better with quotes
echo "The time is $(date +%H:%M)"
```

### Assigning to Variables

```bash
# Capture command output
current_date=$(date +%Y-%m-%d)
hostname=$(hostname)
user_count=$(wc -l < /etc/passwd)

# Use captured values
echo "Date: $current_date"
echo "Host: $hostname"
echo "Users: $user_count"

# Multi-line output
file_list=$(ls -la)
echo "$file_list"    # Preserves newlines with quotes
echo $file_list      # Collapses to single line without quotes
```

### In-Line Usage

```bash
# Build filenames dynamically
backup_file="backup_$(date +%Y%m%d).tar.gz"
log_file="/var/log/app_$(hostname).log"

# Pass as arguments
mkdir "project_$(date +%Y%m%d)"
grep "error" $(find . -name "*.log")

# Combine with other text
echo "Uptime: $(uptime -p)"
echo "Disk usage: $(df -h / | tail -1 | awk '{print $5}')"
```

### Arithmetic with Command Substitution

```bash
# Combine with arithmetic
file_count=$(ls | wc -l)
double=$((file_count * 2))

# Or directly
echo "Files: $(ls | wc -l)"
echo "Double: $(($(ls | wc -l) * 2))"
```

### Nested Substitution

```bash
# Nesting is clean with $()
result=$(cat $(ls *.txt | head -1))

# Multiple levels
owner=$(stat -c %U $(which bash))
echo "Bash is owned by: $owner"
```

### Common Patterns

```bash
# Get directory of current script
script_dir=$(dirname "$0")
script_dir=$(cd "$(dirname "$0")" && pwd)  # Absolute path

# Process each line of command output
for file in $(find . -name "*.txt"); do
    echo "Processing: $file"
done

# Better with while for files with spaces
find . -name "*.txt" | while read -r file; do
    echo "Processing: $file"
done

# Conditional based on command output
if [[ $(whoami) == "root" ]]; then
    echo "Running as root"
fi

# Default value if command fails
value=$(command 2>/dev/null || echo "default")
```

### Word Splitting Considerations

```bash
# Unquoted substitution undergoes word splitting
files=$(ls)
for f in $files; do      # Splits on whitespace
    echo "$f"
done

# Quoted preserves as single string
all_files="$(ls)"
echo "$all_files"         # One string with newlines

# Be careful with filenames containing spaces
for f in $(ls); do        # WRONG: breaks on spaces
    ...
done

# Safer approaches
for f in *; do            # Glob expansion
    ...
done
```

## Interview Questions

**Q: What is the difference between `$(command)` and `` `command` ``?**
**A:** Both capture command output, but `$()` is preferred because it nests cleanly without escaping, is more readable, and works better within quotes. Backticks require `\`` for nesting and are considered legacy.

**Q: How does command substitution differ from piping?**
**A:** Piping (`|`) streams output to another command's stdin. Command substitution (`$()`) captures output as a string that can be stored, manipulated, or embedded in another command. Pipes are for data flow; substitution is for capturing.

**Q: What happens to newlines in command substitution?**
**A:** Trailing newlines are removed. Internal newlines are preserved but collapse to spaces if not quoted. Use `"$(command)"` to preserve newlines in the result.

**Q: How do you handle command failure in substitution?**
**A:** Use `||` for default: `result=$(command 2>/dev/null || echo "default")`. Or check exit status: `if output=$(command); then ... fi`. With `set -e`, failed substitution in assignment doesn't exit.

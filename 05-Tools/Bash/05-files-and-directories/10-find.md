---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# find

## Summary

The `find` command searches for files and directories based on various criteria: name, type, size, time, permissions, and more. It's extremely powerful for locating files and performing bulk operations. Mastering find is essential for system administration and scripting.

## Detailed Explanation

### Basic Syntax

```bash
find [path] [expression]

# Search in current directory
find . -name "*.txt"

# Search from root (slow!)
find / -name "config.yaml" 2>/dev/null
```

### Finding by Name

```bash
# Exact name
find . -name "config.yaml"

# Case-insensitive
find . -iname "readme.md"

# Wildcards (must quote!)
find . -name "*.log"
find . -name "test_*"

# Using -path for full path matching
find . -path "*/src/*.go"
```

### Finding by Type

```bash
# -type options:
# f = file, d = directory, l = symlink
# p = pipe, s = socket, b = block, c = char

find . -type f -name "*.py"     # Files only
find . -type d -name "node_*"   # Directories only
find . -type l                   # Symlinks
```

### Finding by Size

```bash
# Size suffixes: c=bytes, k=KB, M=MB, G=GB
find . -size +100M      # Larger than 100MB
find . -size -1k        # Smaller than 1KB
find . -size 10M        # Exactly 10MB (rare)

# Find large files
find / -type f -size +1G 2>/dev/null
```

### Finding by Time

```bash
# -mtime: modification time (days)
find . -mtime -7        # Modified in last 7 days
find . -mtime +30       # Modified more than 30 days ago

# -mmin: modification time (minutes)
find . -mmin -60        # Modified in last hour

# -newer: newer than reference file
find . -newer reference.txt

# Access time: -atime, -amin
# Change time: -ctime, -cmin
```

### Finding by Permissions

```bash
# Exact permissions
find . -perm 755

# At least these permissions
find . -perm -644

# Any of these permissions
find . -perm /222       # Writable by anyone

# Find world-writable files
find . -type f -perm -o+w
```

### Combining Conditions

```bash
# AND (implicit)
find . -type f -name "*.log" -size +1M

# OR
find . -name "*.txt" -o -name "*.md"

# NOT
find . ! -name "*.txt"

# Grouping (escape parentheses)
find . \( -name "*.js" -o -name "*.ts" \) -type f
```

### Executing Commands

```bash
# -exec: run command on each file
find . -name "*.tmp" -exec rm {} \;

# -exec with +: batch (faster)
find . -name "*.txt" -exec grep -l "pattern" {} +

# -delete: built-in delete (faster than -exec rm)
find . -name "*.bak" -delete

# -print0 + xargs -0: handle spaces in filenames
find . -name "*.txt" -print0 | xargs -0 wc -l
```

### Practical Examples

```bash
# Find and delete old logs
find /var/log -name "*.log" -mtime +30 -delete

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Find recently modified
find . -type f -mmin -30

# Find empty directories
find . -type d -empty

# Find and replace in files
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# Find broken symlinks
find . -xtype l
```

## Interview Questions

**Q: What is the difference between `-exec {} \;` and `-exec {} +`?**
**A:** `\;` runs the command once per file found. `+` batches files and runs fewer commands, which is faster. Use `+` when the command can accept multiple arguments (like `grep`, `ls`).

**Q: How do you find files modified in the last 24 hours?**
**A:** Use `find . -type f -mtime -1` (less than 1 day ago) or `find . -type f -mmin -1440` (less than 1440 minutes).

**Q: How do you handle filenames with spaces in find?**
**A:** Use `-print0` with `xargs -0`: `find . -name "*.txt" -print0 | xargs -0 command`. This uses null bytes as delimiters instead of newlines.

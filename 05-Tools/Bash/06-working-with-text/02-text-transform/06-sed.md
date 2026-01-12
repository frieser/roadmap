---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# sed (Stream Editor)

## Summary

`sed` is a stream editor for transforming text. It reads input line by line, applies editing commands, and outputs the result. Sed is powerful for find/replace, deletions, insertions, and complex text transformations. It's essential for batch editing files and text processing pipelines.

## Detailed Explanation

### Basic Substitution

```bash
# Syntax: s/pattern/replacement/flags
sed 's/old/new/' file.txt       # First occurrence per line
sed 's/old/new/g' file.txt      # All occurrences (global)
sed 's/old/new/gi' file.txt     # Global, case-insensitive

# Edit in place
sed -i 's/old/new/g' file.txt   # Linux
sed -i '' 's/old/new/g' file    # macOS (BSD sed)

# Backup before editing
sed -i.bak 's/old/new/g' file.txt
```

### Address Selection

```bash
# Apply to specific lines
sed '3s/old/new/' file.txt      # Line 3 only
sed '1,5s/old/new/' file.txt    # Lines 1-5
sed '$s/old/new/' file.txt      # Last line

# Pattern address
sed '/error/s/old/new/' file.txt    # Lines containing "error"
sed '/^#/d' file.txt                # Delete comment lines

# Range by pattern
sed '/START/,/END/d' file.txt       # Delete between patterns
```

### Common Operations

```bash
# Delete lines
sed 'd' file.txt                # Delete all
sed '5d' file.txt               # Delete line 5
sed '1,10d' file.txt            # Delete lines 1-10
sed '/pattern/d' file.txt       # Delete matching lines
sed '/^$/d' file.txt            # Delete empty lines

# Print (with -n for quiet mode)
sed -n 'p' file.txt             # Print all
sed -n '5p' file.txt            # Print line 5
sed -n '/error/p' file.txt      # Print matching lines

# Insert and append
sed '3i\New line before' file.txt   # Insert before line 3
sed '3a\New line after' file.txt    # Append after line 3
sed '$a\Last line' file.txt         # Append at end
```

### Advanced Substitution

```bash
# Delimiter alternatives (when pattern has /)
sed 's|/usr/local|/opt|g' file.txt
sed 's#http://#https://#g' file.txt

# Capture groups
sed 's/\(.*\):\(.*\)/\2:\1/' file.txt   # Swap around :
sed -E 's/(.*):(.*)/\2:\1/' file.txt    # Extended regex

# Case conversion (GNU sed)
sed 's/\(.*\)/\U\1/' file.txt          # Uppercase
sed 's/\(.*\)/\L\1/' file.txt          # Lowercase

# & references entire match
sed 's/[0-9]*/(&)/' file.txt           # Wrap numbers in ()
```

### Multiple Commands

```bash
# -e for multiple expressions
sed -e 's/a/A/' -e 's/b/B/' file.txt

# Semicolon separator
sed 's/a/A/; s/b/B/' file.txt

# From file
sed -f commands.sed file.txt

# Newline in replacement
sed 's/:/:\n/g' file.txt
```

### Practical Examples

```bash
# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Add prefix to lines
sed 's/^/PREFIX: /' file.txt

# Extract between patterns
sed -n '/START/,/END/p' file.txt

# Replace in specific file types
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# Remove HTML tags
sed 's/<[^>]*>//g' file.html
```

## Interview Questions

**Q: What is the difference between `sed 's/old/new/'` and `sed 's/old/new/g'`?**
**A:** Without `g`, sed replaces only the first occurrence on each line. With `g` (global), it replaces all occurrences on each line.

**Q: How do you edit a file in place with sed?**
**A:** Use `sed -i 's/old/new/g' file`. On macOS, use `sed -i '' 's/old/new/g' file`. Create a backup with `sed -i.bak`.

**Q: How do you delete lines matching a pattern?**
**A:** Use `sed '/pattern/d' file`. For example, `sed '/^#/d'` removes comment lines, `sed '/^$/d'` removes empty lines.

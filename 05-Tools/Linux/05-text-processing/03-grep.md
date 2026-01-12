---
tags: ['linux', 'roadmap']
---

# grep (Global Regular Expression Print)

## Summary
`grep` is a powerful command-line utility used for searching plain-text data sets for lines that match a regular expression. It is one of the most essential tools in the Unix/Linux ecosystem, derived from the `ed` editor command `g/re/p` (global / regular expression / print). It is commonly used for filtering logs, searching through codebases, and processing command output in pipelines.

## Detailed Explanation

### Basic Syntax
The basic syntax for `grep` is:
```bash
grep [options] pattern [files]
```
If no file is specified, `grep` reads from standard input (`stdin`).

### Common Flags
- `-i` (**Ignore Case**): Performs case-insensitive matching.
- `-v` (**Invert Match**): Displays lines that do NOT match the pattern.
- `-r` or `-R` (**Recursive**): Searches through directories and their subdirectories.
- `-n` (**Line Number**): Prefixes each matching line with its line number within the file.
- `-w` (**Word Regexp**): Matches only whole words (e.g., searching for "cat" won't match "category").
- `-l` (**Files with Matches**): Prints only the names of files containing a match, not the matching lines themselves.
- `-c` (**Count**): Returns the number of lines that match the pattern instead of the lines themselves.
- `-o` (**Only Matching**): Prints only the matched parts of a line, each on a separate output line.

### Context Control
Sometimes it is useful to see the lines surrounding a match:
- `-A [n]` (**After**): Prints `n` lines of context after the match.
- `-B [n]` (**Before**): Prints `n` lines of context before the match.
- `-C [n]` (**Context**): Prints `n` lines of context both before and after the match.

### Regular Expression Support
`grep` supports different levels of regular expressions:
1. **Basic Regular Expressions (BRE)**: The default. Meta-characters like `|`, `+`, `?`, `(`, and `)` must be escaped (`\|`, `\+`, etc.) to be treated as special.
2. **Extended Regular Expressions (ERE)**: Enabled with `-E` (or by using `egrep`). Meta-characters do not need escaping.
3. **Perl-Compatible Regular Expressions (PCRE)**: Enabled with `-P`. Provides the most advanced features like lookaheads and non-greedy matching.

### Bash Examples

```bash
# Search for "error" in a specific file (case-insensitive)
grep -i "error" /var/log/syslog

# Search recursively for "TODO" in the current directory
grep -ri "TODO" .

# Filter running processes for a specific application
ps aux | grep "nginx"

# Find lines NOT containing "DEBUG" in a log file
grep -v "DEBUG" application.log

# Show match with 3 lines of context before and after
grep -C 3 "critical_failure" server.log

# Count occurrences of an IP address in access logs
grep -c "192.168.1.1" access.log
```

## Interview Questions

**Q: How do you search for a string recursively in all files within a directory?**
**A:** Use the `-r` (recursive) flag: `grep -r "pattern" /path/to/directory`. You can use `-i` as well if you want the search to be case-insensitive.

**Q: What is the difference between `grep`, `egrep`, and `fgrep`?**
**A:** `grep` uses Basic Regular Expressions (BRE). `egrep` (equivalent to `grep -E`) uses Extended Regular Expressions (ERE). `fgrep` (equivalent to `grep -F`) treats the pattern as a fixed string rather than a regular expression, making it faster for simple searches.

**Q: How can you display the line numbers along with the matched lines?**
**A:** Use the `-n` flag: `grep -n "pattern" file.txt`.

**Q: How do you find lines that do NOT match a specific pattern?**
**A:** Use the `-v` (invert-match) flag: `grep -v "exclude_this" file.log`.

**Q: How do you match a string only when it forms a complete word?**
**A:** Use the `-w` flag: `grep -w "is" file.txt`. This will match "is" but not "island" or "thesis".

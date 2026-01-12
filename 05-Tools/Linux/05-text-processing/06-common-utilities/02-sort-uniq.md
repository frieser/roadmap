---
tags: ['linux', 'roadmap']
---

# sort and uniq commands

## Summary
The `sort` and `uniq` commands are fundamental Linux utilities for organizing and filtering text data. `sort` reorders lines based on alphabetical, numerical, or field-specific criteria, while `uniq` identifies and processes repeated lines. Together, they form a powerful pipeline for data analysis, log processing, and deduplication tasks in the shell.

## Detailed Explanation

### The `sort` Command
The `sort` command organizes the lines of a text file or standard input. By default, it sorts in ascending alphabetical order.

#### Common Options:
- **`-n` (Numeric Sort)**: Compares according to string numerical value. Without this, `10` would come before `2`.
- **`-r` (Reverse)**: Reverses the result of the comparison.
- **`-k` (Key Definition)**: Sorts by a specific field. Fields are determined by a delimiter (default is whitespace).
    - Format: `-k POS1[,POS2]` (e.g., `-k 2,2` sorts by the 2nd field only).
- **`-t` (Field Separator)**: Specifies a character to use as the field delimiter (e.g., `-t ','` for CSV).
- **`-u` (Unique)**: With the `-u` option, `sort` outputs only the first of an equal run. It is often faster than `sort | uniq`.
- **`-M` (Month Sort)**: Sorts by month names (JAN, FEB, etc.).
- **`-h` (Human-numeric Sort)**: Sorts values like `2K`, `1G`, `10M`.

#### Bash Examples:
```bash
# Sort numbers numerically in descending order
echo -e "15\n2\n100\n45" | sort -nr

# Sort /etc/passwd by UID (3rd field) using ':' as delimiter
sort -t: -k3 -n /etc/passwd | head -n 5

# Sort a list of file sizes from 'ls -lh'
ls -lh | awk '{print $5, $9}' | sort -h
```

---

### The `uniq` Command
The `uniq` command filters out repeated lines. **Crucially, `uniq` only detects duplicate lines that are adjacent.** This is why it is almost always used in combination with `sort`.

#### Common Options:
- **`-c` (Count)**: Prefixes lines by the number of occurrences.
- **`-d` (Repeated)**: Only prints duplicate lines.
- **`-u` (Unique)**: Only prints lines that are not repeated (true uniqueness).
- **`-i` (Ignore Case)**: Performs case-insensitive comparison.
- **`-f N` (Skip Fields)**: Avoids comparing the first N fields.

#### Bash Examples:
```bash
# Count occurrences of each word in a list
echo -e "blue\nred\nblue\ngreen\nred\nred" | sort | uniq -c

# Find only the lines that appear more than once
cat data.txt | sort | uniq -d

# Show only lines that are unique (never repeated)
cat names.txt | sort | uniq -u
```

### Powerful Pipelines
Combining these tools allows for quick data profiling.

```bash
# Top 10 most frequent IP addresses in a web server log
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 10

# List unique extensions in a directory
find . -type f | awk -F. '{if (NF>1) print $NF}' | sort -u
```

## Interview Questions

**Q: Why must you usually pipe the output of `sort` into `uniq`?**
**A:** `uniq` only compares consecutive lines. If the data is not sorted, duplicate lines that are separated by other text will not be detected. Sorting brings all identical lines together, making them adjacent for `uniq`.

**Q: How do you sort a file by its second column numerically, using a comma as a separator?**
**A:** Use the command `sort -t',' -k2 -n filename`. `-t` sets the separator, `-k2` targets the second field, and `-n` ensures numeric evaluation.

**Q: What is the difference between `sort -u` and `sort | uniq`?**
**A:** `sort -u` is often more performance-efficient as it handles deduplication during the sort process. However, `sort | uniq` is necessary if you need the extra features of `uniq`, such as counting occurrences (`-c`) or finding only duplicates (`-d`).

**Q: How can you find the most frequent line in a text file?**
**A:** By using the pipeline: `sort | uniq -c | sort -nr | head -n 1`. This sorts the data, counts occurrences, sorts those counts numerically in reverse, and takes the top result.

**Q: How would you perform a case-insensitive unique count?**
**A:** Use `sort -f | uniq -ci`. The `-f` in sort ignores case during sorting, and `-i` in uniq ignores case when comparing adjacent lines.

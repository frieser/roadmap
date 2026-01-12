#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
The `cut` and `paste` commands are fundamental Linux utilities for vertical and horizontal text manipulation. `cut` extracts specific columns or fields from a file based on delimiters, characters, or bytes. `paste` performs the inverse operation, merging lines from multiple files side-by-side or serially. Together, they are essential tools in the Unix philosophy for processing tabular data and text streams without complex scripting.

## Detailed Explanation

### The `cut` Command
The `cut` command is used to remove sections from each line of files. It is most commonly used to extract fields from structured files like `/etc/passwd` or CSVs.

**Key Options:**
- `-d <delimiter>`: Specifies the field delimiter (defaults to TAB).
- `-f <list>`: Selects specific fields (e.g., `1`, `1,3`, `1-5`).
- `-c <list>`: Selects specific character positions.
- `-b <list>`: Selects specific byte positions.
- `--complement`: Extracts everything *except* the specified fields/characters.
- `--output-delimiter=<string>`: Changes the delimiter in the output.

**Bash Examples:**
```bash
# Extract the first and third fields from /etc/passwd (colon-delimited)
cut -d':' -f1,3 /etc/passwd

# Extract characters 1 to 10 of each line
cut -c1-10 file.txt

# Extract all fields EXCEPT the second one (comma-delimited)
cut -d',' -f2 --complement data.csv

# Change the output delimiter from colon to space
cut -d':' -f1,3 --output-delimiter=' ' /etc/passwd
```

### The `paste` Command
The `paste` command merges lines from multiple files into a single stream, placing them side-by-side. It is the horizontal counterpart to `cat`.

**Key Options:**
- `-d <delimiter>`: Specifies the delimiter to use between merged lines (defaults to TAB).
- `-s`: Merges lines serially from one file (transforms columns to rows).

**Bash Examples:**
```bash
# Merge two files side-by-side separated by a tab
paste file1.txt file2.txt

# Merge two files using a comma as the delimiter
paste -d',' names.txt ages.txt

# Convert all lines in a file into a single comma-separated line (Serial mode)
paste -s -d',' list.txt

# Join lines in pairs (using standard input)
cat numbers.txt | paste - -
```

### Combining `cut` and `paste`
These tools are often used in pipelines to reorganize data. For instance, you can swap columns by cutting them into temporary streams and pasting them back in a different order.

```bash
# Swap the first and second columns of a space-separated file
paste <(cut -d' ' -f2 file.txt) <(cut -d' ' -f1 file.txt)
```

## Interview Questions

**Q: How do you extract the last field of a line if you don't know the total number of fields using `cut`?**
**A:** The `cut` command does not natively support indexing from the end (unlike `awk`'s `NF`). A common workaround is to reverse the string, cut the first field, and reverse it back: `rev file.txt | cut -d',' -f1 | rev`.

**Q: What is the difference between `cut -c` and `cut -b`?**
**A:** `cut -c` works with **characters**, while `cut -b` works with **bytes**. In multi-byte encodings like UTF-8, a character might be 2-4 bytes. `cut -c` is generally safer for text containing international characters (emojis, accented letters), while `cut -b` is faster for fixed-width ASCII data.

**Q: How can you use `paste` to join every 3 lines into a single line?**
**A:** You can pass the hyphen `-` (representing stdin) multiple times to `paste`: `cat file.txt | paste - - -`.

**Q: How do you handle files with multiple delimiters in `cut`?**
**A:** `cut` only supports a single character as a delimiter. If your file uses multiple spaces or tabs as delimiters, it's better to use `tr -s ' '` to squeeze spaces first, or use `awk`, which handles variable whitespace by default.

**Q: What happens if you use `paste -s` on multiple files?**
**A:** Instead of merging the first lines of all files, `paste -s` will take the entire content of the first file and put it on one line, then the entire content of the second file on the next line, and so on.

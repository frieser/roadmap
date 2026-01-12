---
tags: ['linux', 'roadmap']
---

# awk

## Summary
`awk` is a versatile, record-oriented programming language designed for text processing, data extraction, and reporting. It is particularly effective at handling structured data arranged in columns (fields) and rows (records), making it a staple for command-line data manipulation and log analysis in Unix-like environments.

## Detailed Explanation

### Core Concepts: Patterns and Actions
An `awk` program consists of a series of **pattern-action** pairs:
```bash
awk 'pattern { action }' input_file
```
- If the **pattern** matches, the **action** is executed.
- If no pattern is provided, the action runs for every line.
- The default action is `{ print $0 }` (print the entire line).

### Fields and Records
`awk` views input as a sequence of records (usually lines) composed of fields (usually words).
- `$0`: Represents the entire current record.
- `$1, $2, ... $n`: Represent the first, second, through n-th fields.
- `NF`: A built-in variable containing the **Number of Fields** in the current record.
- `NR`: A built-in variable containing the **Number of Records** (line count) processed so far.

**Example: Printing specific columns**
```bash
# Print the first and third columns of a file
awk '{ print $1, $3 }' data.txt
```

### Changing the Delimiter
By default, `awk` uses any whitespace (space or tab) as a field separator. Use the `-F` flag to specify a different one.
```bash
# Process /etc/passwd using colon as a delimiter
awk -F':' '{ print $1, $6 }' /etc/passwd
```

### BEGIN and END Blocks
Special patterns that trigger actions before or after processing the input:
- `BEGIN`: Executed once before any input lines are read. Used for setup (e.g., setting separators, printing headers).
- `END`: Executed once after all input lines are processed. Used for summaries or totals.

**Example: Summing a column**
```bash
# Sum the values in the second column
awk '{ sum += $2 } END { print "Total Sum:", sum }' numbers.txt
```

### Built-in Variables and Functions
- `FS`: Input Field Separator (variable version of `-F`).
- `OFS`: Output Field Separator (defaults to space).
- `length($0)`: Returns the number of characters in the record.
- `substr($1, 1, 3)`: Returns a substring of the first field.

## Interview Questions

1. **What is the default field separator in `awk`, and how can you change it?**
   - The default is any whitespace (spaces or tabs). You can change it using the `-F` option on the command line or by setting the `FS` variable in a `BEGIN` block.

2. **Explain the difference between `$0` and `NF`.**
   - `$0` refers to the entire current line/record being processed, while `NF` is a variable that stores the total number of fields found in that line.

3. **How would you print only the lines that have more than 5 fields?**
   - Use the pattern `NF > 5`: `awk 'NF > 5' file.txt`.

4. **How can you pass a shell variable into an `awk` script?**
   - Use the `-v` option: `awk -v myvar="$SHELL_VAR" '{ print myvar, $1 }' file.txt`.

5. **What is `FNR` and how does it differ from `NR`?**
   - `NR` is the total number of records processed across all files in the current `awk` execution. `FNR` is the record number relative to the *current* file being processed (resets to 1 for each new file).

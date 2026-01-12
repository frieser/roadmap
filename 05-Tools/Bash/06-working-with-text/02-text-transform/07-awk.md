---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# awk

## Summary

`awk` is a powerful pattern-scanning and processing language. It excels at field-based text processing, making it ideal for working with structured data like CSV, logs, and tabular output. Awk can perform complex text transformations, calculations, and report generation.

## Detailed Explanation

### Basic Syntax

```bash
awk 'pattern { action }' file

# Default action is print
awk '/error/' file.txt          # Print lines containing "error"

# Default pattern is all lines
awk '{ print }' file.txt        # Print all lines

# Multiple patterns/actions
awk '/start/ { print "Begin" } /end/ { print "Finish" }' file
```

### Fields and Records

```bash
# Fields: $1, $2, ... $NF (last field)
# $0 = entire line
# NF = number of fields
# NR = record (line) number

awk '{ print $1 }' file.txt           # First field
awk '{ print $1, $3 }' file.txt       # First and third
awk '{ print $NF }' file.txt          # Last field
awk '{ print NR, $0 }' file.txt       # Line numbers

# Field separator
awk -F: '{ print $1 }' /etc/passwd    # Split on :
awk -F, '{ print $2 }' data.csv       # Split on ,
awk -F'\t' '{ print $1 }' file.tsv    # Split on tab
```

### Pattern Matching

```bash
# Regex patterns
awk '/^error/' file.txt               # Lines starting with "error"
awk '!/comment/' file.txt             # Lines NOT matching

# Comparison
awk '$3 > 100' file.txt               # Third field > 100
awk '$1 == "admin"' file.txt          # First field equals "admin"
awk 'NR > 1' file.txt                 # Skip header (line 1)

# Range patterns
awk '/START/,/END/' file.txt          # Between patterns
awk 'NR==5,NR==10' file.txt           # Lines 5-10
```

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `$0` | Entire line |
| `$1..$N` | Fields |
| `NF` | Number of fields |
| `NR` | Record number (line) |
| `FS` | Field separator |
| `OFS` | Output field separator |
| `RS` | Record separator |
| `ORS` | Output record separator |

### Actions and Calculations

```bash
# Arithmetic
awk '{ sum += $1 } END { print sum }' numbers.txt
awk '{ print $1 * $2 }' file.txt

# String operations
awk '{ print length($0) }' file.txt   # Line length
awk '{ print toupper($1) }' file.txt  # Uppercase
awk '{ gsub(/old/, "new"); print }' file.txt

# Conditional
awk '{ if ($1 > 100) print "big"; else print "small" }' file.txt

# BEGIN and END blocks
awk 'BEGIN { print "Header" } { print } END { print "Footer" }' file
```

### Practical Examples

```bash
# Sum a column
awk '{ sum += $1 } END { print sum }' numbers.txt

# Average
awk '{ sum += $1; count++ } END { print sum/count }' numbers.txt

# Print columns in new order
awk '{ print $3, $1, $2 }' file.txt

# Filter and format
awk -F: '$3 >= 1000 { print $1 }' /etc/passwd

# Count occurrences
awk '{ count[$1]++ } END { for (word in count) print word, count[word] }' file.txt

# CSV processing
awk -F, 'NR > 1 { sum += $3 } END { print "Total:", sum }' data.csv

# Log analysis - requests per IP
awk '{ ips[$1]++ } END { for (ip in ips) print ip, ips[ip] }' access.log | sort -k2 -rn

# Transpose columns
awk '{ for(i=1; i<=NF; i++) a[NR,i]=$i } END { for(i=1; i<=NF; i++) { for(j=1; j<=NR; j++) printf a[j,i] " "; print "" } }' file
```

### printf Formatting

```bash
# Formatted output
awk '{ printf "%-10s %5d\n", $1, $2 }' file.txt

# Format specifiers
# %s = string
# %d = integer  
# %f = float
# %-10s = left-align, 10 chars wide
```

## Interview Questions

**Q: What is the difference between `$0` and `$1` in awk?**
**A:** `$0` is the entire line/record. `$1` is the first field (word) of the line. Fields are separated by whitespace by default, changeable with `-F`.

**Q: How do you sum a column of numbers with awk?**
**A:** `awk '{ sum += $1 } END { print sum }' file.txt`. The END block runs after processing all lines, printing the accumulated sum.

**Q: How do you process CSV files with awk?**
**A:** Set the field separator: `awk -F, '{ print $2 }' file.csv`. For CSVs with quoted fields containing commas, consider using specialized tools like csvtool.

---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Input Redirection

## Summary

Input redirection in Bash allows commands to read data from files instead of the keyboard. The `<` operator redirects standard input (stdin) from a file. This enables processing files with commands that normally expect interactive input, and is essential for automation and batch processing.

## Detailed Explanation

### Basic Input Redirection

```bash
# Read input from file instead of keyboard
wc -l < file.txt          # Count lines in file
sort < unsorted.txt       # Sort file contents
tr 'a-z' 'A-Z' < input.txt # Convert to uppercase

# Compare: with and without redirection
cat file.txt              # cat opens the file itself
cat < file.txt            # Shell opens file, passes to cat's stdin

# The difference matters for some commands
# Using <, the command doesn't "know" the filename
```

### Input Redirection vs Arguments

```bash
# These produce the same output but work differently:

# 1. File as argument (command opens file)
sort file.txt

# 2. Input redirection (shell opens file, feeds stdin)
sort < file.txt

# Key difference: with redirection, command sees stdin, not filename
wc file.txt          # Outputs: "10 file.txt"
wc < file.txt        # Outputs: "10" (no filename)
```

### Combining Input and Output Redirection

```bash
# Input from file, output to file
sort < unsorted.txt > sorted.txt

# Process file in-place (careful!)
# This WON'T work (file gets truncated first):
sort < file.txt > file.txt   # WRONG!

# Correct approaches:
sort file.txt > temp.txt && mv temp.txt file.txt
sort -o file.txt file.txt    # sort has -o option
```

### Reading Files in Scripts

```bash
#!/bin/bash

# Read file line by line
while IFS= read -r line; do
    echo "Processing: $line"
done < input.txt

# Read into array
mapfile -t lines < file.txt
echo "Line count: ${#lines[@]}"

# Read with custom delimiter
while IFS=: read -r user pass uid gid name home shell; do
    echo "User: $user, Home: $home"
done < /etc/passwd
```

### Here Documents (<<)

```bash
# Inline input block (covered in detail separately)
cat << EOF
Line 1
Line 2
Line 3
EOF

# Useful for multi-line input to commands
mysql -u root << SQL
SELECT * FROM users;
UPDATE stats SET count = count + 1;
SQL
```

### Here Strings (<<<)

```bash
# Single-line input from string
grep "pattern" <<< "search in this string"

# Variable as input
data="Hello World"
wc -w <<< "$data"    # Outputs: 2

# Alternative to echo | command
echo "$data" | wc -w    # Same result, extra process
wc -w <<< "$data"       # More efficient
```

### Practical Examples

```bash
# Process configuration file
while IFS='=' read -r key value; do
    case "$key" in
        username) USERNAME="$value" ;;
        password) PASSWORD="$value" ;;
    esac
done < config.ini

# Send email with body from file
mail -s "Report" user@example.com < report.txt

# Feed input to interactive command
ftp -n << EOF
open ftp.example.com
user anonymous
get file.txt
bye
EOF
```

## Interview Questions

**Q: What is the difference between `command file.txt` and `command < file.txt`?**
**A:** With `command file.txt`, the command opens and reads the file itself. With `command < file.txt`, the shell opens the file and provides its contents as stdin. The command doesn't know the filename - it just receives data on stdin.

**Q: How do you read a file line by line in Bash?**
**A:** Use `while IFS= read -r line; do ... done < file.txt`. The `IFS=` preserves leading/trailing whitespace, `-r` prevents backslash interpretation. This is safer than `for line in $(cat file)` which has word-splitting issues.

**Q: What is a here string?**
**A:** A here string (`<<<`) passes a string as stdin to a command. For example, `grep pattern <<< "$variable"` is more efficient than `echo "$variable" | grep pattern` because it avoids spawning a subshell.

**Q: Why is `sort < file.txt > file.txt` dangerous?**
**A:** The shell processes redirections before running the command. `> file.txt` truncates the file first, so by the time sort reads from it, it's empty. Use `sort -o file.txt file.txt` or a temporary file.

---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Strings in Bash

## Summary

Strings are the fundamental data type in Bash - essentially, everything is treated as a string by default. Understanding string declaration, quoting rules, and manipulation is essential for effective shell scripting. Bash provides no native string type declaration; variables simply hold text that can be manipulated using parameter expansion, external commands, or built-in string operations.

## Detailed Explanation

### String Declaration and Assignment

```bash
# Basic string assignment (no spaces around =)
name="John Doe"
greeting='Hello World'
empty=""

# Without quotes (single word only, no spaces)
simple=hello

# Command substitution into string
current_date="Today is $(date +%Y-%m-%d)"
files_count="Found $(ls | wc -l) files"
```

### Quoting Rules: Single vs Double Quotes

```bash
name="Alice"
path="/home/user"

# Double quotes: variable expansion happens
echo "Hello, $name"          # Output: Hello, Alice
echo "Path is: $path"        # Output: Path is: /home/user

# Single quotes: literal string, no expansion
echo 'Hello, $name'          # Output: Hello, $name
echo 'Path is: $path'        # Output: Path is: $path

# Escaping special characters in double quotes
echo "She said \"Hello\""    # Output: She said "Hello"
echo "Cost: \$100"           # Output: Cost: $100
echo "Line1\nLine2"          # Output: Line1\nLine2 (literal \n)

# For actual newline, use $'...' syntax
echo $'Line1\nLine2'         # Output: Line1
                             #         Line2
```

### String Concatenation

```bash
first="Hello"
second="World"

# Simple concatenation (juxtaposition)
combined="$first $second"
echo "$combined"             # Output: Hello World

# Concatenation without spaces
prefix="file"
suffix=".txt"
filename="$prefix$suffix"    # file.txt

# Using curly braces for clarity
version="2"
echo "app_v${version}_final" # app_v2_final

# Appending to a string
message="Hello"
message+=" World"            # Hello World
message+=", how are you?"    # Hello World, how are you?
```

### String Length

```bash
text="Hello World"

# Using parameter expansion
echo "${#text}"              # Output: 11

# Store in variable
length="${#text}"

# Empty string check
empty=""
if [[ ${#empty} -eq 0 ]]; then
    echo "String is empty"
fi
```

### Substring Extraction

```bash
string="Hello World"

# Extract from position (0-indexed)
echo "${string:0:5}"         # Output: Hello (start:length)
echo "${string:6}"           # Output: World (from position 6 to end)
echo "${string:6:3}"         # Output: Wor

# Negative indexing (from end)
echo "${string: -5}"         # Output: World (space before - is required!)
echo "${string: -5:3}"       # Output: Wor

# Extract all but last N characters
echo "${string:0:-3}"        # Output: Hello Wo (all except last 3)
```

### String Replacement

```bash
text="hello world world"

# Replace first occurrence
echo "${text/world/universe}"     # hello universe world

# Replace all occurrences
echo "${text//world/universe}"    # hello universe universe

# Replace at beginning (prefix)
echo "${text/#hello/hi}"          # hi world world

# Replace at end (suffix)
echo "${text/%world/planet}"      # hello world planet

# Delete pattern (replace with nothing)
echo "${text//world/}"            # hello  (note: double space)
```

### Case Conversion (Bash 4.0+)

```bash
text="Hello World"

# Convert to lowercase
echo "${text,,}"             # hello world
echo "${text,}"              # hello World (first char only)

# Convert to uppercase
echo "${text^^}"             # HELLO WORLD
echo "${text^}"              # Hello World (first char only)

# Toggle case
echo "${text~~}"             # hELLO wORLD
```

### Pattern Removal

```bash
filename="/path/to/file.txt"

# Remove shortest match from beginning
echo "${filename#*/}"        # path/to/file.txt

# Remove longest match from beginning
echo "${filename##*/}"       # file.txt (basename)

# Remove shortest match from end
echo "${filename%.*}"        # /path/to/file (remove extension)

# Remove longest match from end
echo "${filename%%/*}"       # (empty - removes everything after first /)

# Practical examples
filepath="/home/user/documents/report.tar.gz"
echo "${filepath##*/}"       # report.tar.gz (filename)
echo "${filepath%/*}"        # /home/user/documents (directory)
echo "${filepath%.tar.gz}"   # /home/user/documents/report
echo "${filepath%%.*}"       # /home/user/documents/report
```

### Default Values and Checks

```bash
# Use default if variable is unset or empty
echo "${name:-Anonymous}"    # Anonymous if $name is empty/unset

# Assign default if variable is unset or empty
echo "${name:=Anonymous}"    # Sets $name to Anonymous if empty

# Error if variable is unset or empty
echo "${name:?Error: name is required}"

# Use alternate value if variable IS set
echo "${name:+Hello $name}"  # Prints "Hello $name" only if set
```

### Multiline Strings

```bash
# Using heredoc
message=$(cat <<EOF
This is a multiline
string that preserves
formatting and $variables
EOF
)

# Using $'...' for escape sequences
multiline=$'Line 1\nLine 2\nLine 3'

# Using literal newlines in quotes
text="First line
Second line
Third line"
```

## Interview Questions

### Q1: What is the difference between single and double quotes in Bash?
**A:** Double quotes allow variable expansion and command substitution (`$var`, `$(cmd)`), while single quotes treat everything literally. Use double quotes when you need variable interpolation, single quotes for literal strings.

### Q2: How do you get the length of a string in Bash?
**A:** Use parameter expansion `${#variable}`. For example: `str="hello"; echo ${#str}` outputs `5`.

### Q3: How do you extract a substring in Bash?
**A:** Use `${string:start:length}` syntax. For example: `${text:0:5}` extracts 5 characters starting at position 0. Omit length to get everything from start to end.

### Q4: What does `${var:-default}` do?
**A:** Returns `default` if `$var` is unset or empty, otherwise returns the value of `$var`. It doesn't modify the variable. Use `:=` instead to also assign the default.

### Q5: How do you replace all occurrences of a pattern in a string?
**A:** Use `${variable//pattern/replacement}` with double slashes. Single slash (`${variable/pattern/replacement}`) replaces only the first occurrence.

### Q6: What is the difference between `${var#pattern}` and `${var##pattern}`?
**A:** `#` removes the shortest match of pattern from the beginning, `##` removes the longest match. Similarly, `%` and `%%` work from the end. Common use: `${path##*/}` extracts filename from path.

### Q7: How do you convert a string to lowercase in Bash?
**A:** Use `${variable,,}` for full lowercase or `${variable,}` for first character only (Bash 4.0+). For uppercase, use `^^` and `^` respectively.

### Q8: Why must there be no spaces around the `=` in variable assignment?
**A:** Bash interprets `var = value` as running command `var` with arguments `=` and `value`. The `=` must directly touch both the variable name and value: `var=value`.

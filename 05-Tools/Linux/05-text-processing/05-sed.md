---
tags: ['linux', 'roadmap', 'tools']
---

# sed (stream editor)

## Summary
`sed` (stream editor) is a non-interactive command-line text editor used for performing basic text transformations on an input stream (a file or input from a pipeline). It is exceptionally efficient for tasks like search-and-replace, line deletion, and selective printing, making it a cornerstone of Linux shell scripting and text processing.

## Detailed Explanation

### Basic Syntax
The general form of a `sed` command is:
```bash
sed [options] 'command' file
```

### Substitution (`s`)
Substitution is the most common use case for `sed`.
*   **Basic substitution**: Replaces the first occurrence per line.
    ```bash
    sed 's/unix/linux/' file.txt
    ```
*   **Global substitution**: Replaces all occurrences per line using the `g` flag.
    ```bash
    sed 's/unix/linux/g' file.txt
    ```
*   **Case-insensitive**: Use the `I` flag (GNU extension).
    ```bash
    sed 's/unix/linux/gI' file.txt
    ```

### In-place Editing (`-i`)
To modify the file directly instead of printing to standard output, use the `-i` flag.
```bash
# Direct edit
sed -i 's/localhost/127.0.0.1/g' config.yaml

# Edit with backup (creates config.yaml.bak)
sed -i.bak 's/localhost/127.0.0.1/g' config.yaml
```

### Deleting Lines (`d`)
`sed` can delete lines based on line numbers or patterns.
```bash
# Delete the 3rd line
sed '3d' file.txt

# Delete lines 1 to 5
sed '1,5d' file.txt

# Delete the last line
sed '$d' file.txt

# Delete lines matching a pattern
sed '/DEBUG/d' app.log
```

### Selective Printing (`-n` and `p`)
By default, `sed` prints every line. Use `-n` to suppress this and `p` to print specific matches.
```bash
# Print only lines 10 to 20
sed -n '10,20p' file.txt

# Print only lines containing "ERROR"
sed -n '/ERROR/p' system.log
```

### Using Different Delimiters
If your text contains many slashes (like URLs or paths), you can use any other character as a delimiter.
```bash
# Using # as a delimiter
sed 's#http://#https://#g' urls.txt
```

### Extended Regular Expressions (`-E`)
For modern regex features (like `+`, `?`, or capturing groups `()`), use the `-E` flag.
```bash
# Swap two words
echo "John Doe" | sed -E 's/([A-Z][a-z]+) ([A-Z][a-z]+)/\2, \1/'
# Output: Doe, John
```

## Interview Questions

**Q: How do you replace a string only on a specific line number?**
**A:** You can prefix the command with the line number. For example, `sed '5s/old/new/' file` will only perform the substitution on line 5.

**Q: How do you perform multiple sed commands in a single execution?**
**A:** Use the `-e` flag for each command or separate them with a semicolon.
`sed -e 's/a/b/' -e 's/c/d/' file` or `sed 's/a/b/; s/c/d/' file`.

**Q: How can you delete all empty lines in a file?**
**A:** Use the command `sed '/^$/d' file`. The pattern `^$` matches lines that have nothing between the start (`^`) and end (`$`).

**Q: What is the purpose of the & character in the replacement string?**
**A:** The `&` represents the entire string that was matched by the pattern. For example, `sed 's/[0-9]\+/(&)/' file` wraps any sequence of digits in parentheses.

**Q: How do you insert a line before or after a match?**
**A:** Use `i` (insert before) or `a` (append after).
`sed '/pattern/i New Line Above' file`
`sed '/pattern/a New Line Below' file`

---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# echo

## Summary

The `echo` command outputs text to standard output. It's fundamental for displaying messages, debugging scripts, and generating output. Understanding the differences between `echo` and `printf`, and handling special characters correctly, is essential for shell scripting.

## Detailed Explanation

### Basic Usage

```bash
# Print text
echo "Hello, World!"
echo 'Single quotes work too'
echo Hello without quotes

# Print variable
name="User"
echo "Hello, $name"

# No newline at end
echo -n "No newline"
```

### Options

```bash
# -n: No trailing newline
echo -n "Enter name: "
read name

# -e: Enable escape sequences
echo -e "Line1\nLine2\tTabbed"

# -E: Disable escape sequences (default on most systems)
echo -E "Literal \n"
```

### Escape Sequences (with -e)

```bash
echo -e "Escape sequences:"
echo -e "\n"      # Newline
echo -e "\t"      # Tab
echo -e "\\"      # Backslash
echo -e "\a"      # Alert (bell)
echo -e "\b"      # Backspace
echo -e "\033[31mRed\033[0m"  # Color
```

### echo vs printf

```bash
# echo is simple but less portable
echo "Hello"          # Behavior varies across systems

# printf is more consistent
printf "Hello\n"      # Always needs \n for newline
printf "%s\n" "Hello" # Format string approach

# printf for formatting
printf "%-10s %5d\n" "Name" 42
#          Name    42
```

### Common Patterns

```bash
# Write to file
echo "data" > file.txt
echo "more" >> file.txt

# Redirect to stderr
echo "Error!" >&2

# Empty line
echo

# Multiline with heredoc
cat << 'EOF'
Line 1
Line 2
EOF
```

### Quoting Matters

```bash
# Variables expand in double quotes
echo "Home: $HOME"     # Home: /home/user

# No expansion in single quotes
echo 'Home: $HOME'     # Home: $HOME

# Special characters
echo "Price: \$10"     # Price: $10
echo 'It'\''s easy'    # It's easy
```

## Interview Questions

**Q: What is the difference between `echo` and `printf`?**
**A:** `echo` adds a newline automatically and is simpler but less portable. `printf` requires explicit `\n`, offers format specifiers, and is POSIX-standard for consistent behavior across systems.

**Q: How do you print to stderr with echo?**
**A:** Use `echo "Error message" >&2`. This redirects echo's stdout to stderr (file descriptor 2).

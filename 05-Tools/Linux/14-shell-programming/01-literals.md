---
tags: ['linux', 'roadmap']
---

# Shell Literals (Strings, Numbers, Quoting)

## Summary
Shell literals define how the shell interprets data types like strings and numbers. In Bash, literals are primarily treated as strings, but their interpretation depends heavily on **quoting mechanisms**. Proper quoting is essential to control variable expansion, command substitution, and to prevent unwanted word splitting or globbing.

## Detailed Explanation

### 1. Quoting Mechanisms
Quoting is used to remove the special meaning of certain characters or words to the shell.

#### Single Quotes ('...')
Everything inside single quotes is preserved literally. No expansions (variables, commands, or arithmetic) occur.
```bash
name="Alice"
echo 'Hello $name' 
# Output: Hello $name
```

#### Double Quotes ("...")
Double quotes preserve the literal value of most characters but allow **expansion**:
- Variable expansion: $VAR
- Command substitution: $(command)
- Arithmetic expansion: $((expression))
- Backslash escaping for specific characters: $, `, ", \, and newline.

```bash
name="Alice"
echo "Hello $name" 
# Output: Hello Alice

echo "Today is $(date)"
# Output: Today is [current date]
```

#### Backslash Escaping (\)
A non-quoted backslash preserves the literal value of the next character.
```bash
echo "The cost is \$10.00"
# Output: The cost is $10.00
```

### 2. Number Literals
Bash does not have distinct data types for numbers; variables are essentially strings. However, when used in an **arithmetic context**, Bash interprets strings as integers.

```bash
count="10"
echo $((count + 5)) 
# Output: 15

# Hexadecimal and Octal
hex=0xFF
echo $((hex)) # Output: 255
```

### 3. Here-Documents (<<)
A Here-document (Here-doc) allows you to redirect multiple lines of input to a command.
```bash
cat <<EOF
This is a multi-line string.
Variable expansion works: $USER
EOF
```
*Tip: Using <<'EOF' (quoting the delimiter) prevents expansion inside the block.*

### 4. ANSI-C Quoting ($'...')
Used to represent special characters like newlines, tabs, or unicode characters.
```bash
echo $'Line One\nLine Two'
# Output: 
# Line One
# Line Two
```

## Interview Questions

**Q: What is the primary difference between single and double quotes in Bash?**
**A:** Single quotes preserve the literal value of every character within them, preventing all expansions. Double quotes allow variable expansion ($), command substitution ($()), and arithmetic expansion ($(())), while still protecting most other characters from shell interpretation.

**Q: How do you prevent a variable from undergoing word splitting or globbing?**
**A:** By wrapping the variable reference in double quotes, e.g., "$VARIABLE". This is a best practice to handle variables that might contain spaces or special characters.

**Q: What is a "Here-document" and how do you disable variable expansion within it?**
**A:** A Here-document is a way to pass a multi-line string to a command's standard input. To disable variable expansion, quote the EOF delimiter, like so: cat <<'EOF'.

**Q: How can you perform basic integer arithmetic in a shell script?**
**A:** Using the arithmetic expansion syntax $(( ... )). For example, result=$((5 + 3)).

**Q: What does the <<< operator do?**
**A:** It is a "Here-string," which passes a single string to the standard input of a command, similar to echo "string" | command but more efficient.

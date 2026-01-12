#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Shell conditionals are control structures that allow a script to execute different code blocks based on the success or failure of a command or the evaluation of a conditional expression. They are fundamental for logic implementation, error handling, and automation in shell scripting. The most common forms are the `if` statement, the `case` statement, and the use of the `test` command (often written as `[` or `[[`).

## Detailed Explanation

### 1. The `if` Statement
The `if` statement executes a block of code if the exit status of a command is 0 (success).

```bash
if [ "$condition" ]; then
    # code to execute if true
elif [ "$other_condition" ]; then
    # code to execute if the first was false and this is true
else
    # code to execute if all above were false
fi
```

### 2. `[` vs `[[` (Test vs Compound Command)
While both are used for testing conditions, `[[ ... ]]` is a Bash-specific enhancement over the POSIX-standard `[ ... ]` (which is actually a synonym for the `test` command).

| Feature | `[ ... ]` (test) | `[[ ... ]]` (Bash) |
| :--- | :--- | :--- |
| **Type** | Builtin/External command | Shell keyword |
| **Word Splitting** | Performed on variables | Not performed (safer) |
| **Pattern Matching** | No | Yes (wildcards and `~=` for Regex) |
| **Logical Operators** | `-a` (and), `-o` (or) | `&&` (and), `||` (or) |
| **Redirections** | Requires quoting `<`, `>` | Handled naturally |

**Recommendation**: Use `[[ ... ]]` in Bash scripts for better reliability and features.

### 3. Common Comparison Operators

#### File Tests
Used to check attributes of files and directories.
- `-e file`: True if file **exists**.
- `-f file`: True if file exists and is a **regular file**.
- `-d file`: True if file exists and is a **directory**.
- `-s file`: True if file exists and has a **size > 0**.
- `-r`, `-w`, `-x`: True if file is **readable**, **writable**, or **executable**.
- `-L file`: True if file is a **symbolic link**.

#### String Comparisons
- `"$a" == "$b"`: True if strings are equal.
- `"$a" != "$b"`: True if strings are not equal.
- `-z "$a"`: True if string is **empty** (zero length).
- `-n "$a"`: True if string is **not empty**.

#### Numeric Comparisons
Shell arithmetic comparisons use specific flags rather than symbols (which are often reserved for redirections).
- `-eq`: Equal to
- `-ne`: Not equal to
- `-lt`: Less than
- `-le`: Less than or equal to
- `-gt`: Greater than
- `-ge`: Greater than or equal to

*Example:* `if [ "$count" -gt 10 ]; then ...`

### 4. The `case` Statement
The `case` statement is used for multiple branching based on pattern matching. It is often cleaner than a long `if-elif` chain.

```bash
case "$variable" in
    "pattern1")
        echo "Match 1"
        ;;
    "pattern2" | "pattern3")
        echo "Match 2 or 3"
        ;;
    *)
        echo "Default case (no match)"
        ;;
esac
```

### 5. Logical Operators
- `&&` (AND): Executes the second command only if the first succeeds.
- `||` (OR): Executes the second command only if the first fails.
- `!`: Negates the result of a test.

```bash
# Check if a file exists AND is readable
if [[ -f "$file" && -r "$file" ]]; then
    echo "Ready to read."
fi
```

## Interview Questions

**Q: What is the difference between `[` and `[[` in Bash?**
**A:** `[` is a command (synonym for `test`), while `[[` is a Bash keyword. `[[` is safer because it doesn't perform word splitting or glob expansion on variables, supports regular expression matching (`=~`), and allows the use of `&&` and `||` for logical operations instead of `-a` and `-o`.

**Q: How do you check if a string is empty in a shell script?**
**A:** Use the `-z` operator. For example: `if [[ -z "$my_string" ]]; then echo "String is empty"; fi`. Conversely, `-n` checks if a string is not empty.

**Q: How can you check if a command failed without using an `if` statement?**
**A:** You can use the `||` (OR) operator. For example: `mkdir /tmp/test_dir || echo "Failed to create directory"`. If `mkdir` fails (non-zero exit status), the `echo` command will execute.

**Q: Why do we use `-eq` instead of `==` for comparing numbers in `[`?**
**A:** In the `[` command and `test`, `==` and `=` are used for string comparisons. Using them for numbers might lead to unexpected results if the strings don't match exactly (e.g., "05" vs "5"). `-eq`, `-gt`, etc., explicitly perform numeric evaluation.

**Q: What does the `*` pattern mean in a `case` statement?**
**A:** The `*` acts as a wildcard that matches anything. In a `case` statement, it is used as the final "default" case to catch any values that did not match the preceding patterns.

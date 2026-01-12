---
tags: ['linux', 'roadmap']
---

# Shell Loops (for, while, until)

## Summary
Loops in shell scripting allow for the repetitive execution of a block of commands as long as a certain condition is met or for each item in a list. Bash supports several types of loops: `for` loops for iterating over lists or ranges, `while` loops for executing as long as a condition is true, and `until` loops for executing until a condition becomes true. Mastering loops is essential for automating repetitive tasks like file processing, system monitoring, and log analysis.

## Detailed Explanation

### 1. The `for` Loop
The `for` loop is primarily used to iterate over a list of items or a defined range.

#### Iterating over a List
The most basic form iterates over a space-separated list.
```bash
for name in Alice Bob Charlie; do
    echo "Hello, $name!"
done
```

#### Range-based Loops
You can use brace expansion `{start..end..step}` to define a range.
```bash
for i in {1..10..2}; do
    echo "Number: $i"
done
```

#### C-style `for` Loop
Used for traditional counter-based iteration within double parentheses.
```bash
for ((i=0; i<5; i++)); do
    echo "Iteration $i"
done
```

### 2. The `while` Loop
The `while` loop executes a block of code as long as the specified command or condition evaluates to true (exit status 0).

```bash
counter=1
while [ $counter -le 5 ]; do
    echo "Count: $counter"
    ((counter++))
done
```

#### Reading Lines from a File
One of the most powerful uses of `while` is reading input line by line using the `read` command.
```bash
while IFS= read -r line; do
    echo "Processing line: $line"
done < input.txt
```
*Note: `IFS=` (Internal Field Separator) prevents trimming leading/trailing whitespace, and `-r` prevents backslash escapes from being interpreted.*

### 3. The `until` Loop
The `until` loop is the logical opposite of the `while` loop. It continues execution as long as the condition is **false** (non-zero exit status) and stops when it becomes true.

```bash
counter=1
until [ $counter -gt 5 ]; do
    echo "Until count: $counter"
    ((counter++))
done
```

### 4. Loop Control: `break` and `continue`
- `break`: Immediately exits the loop.
- `continue`: Skips the rest of the current iteration and jumps to the next one.

```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue # Skip 5
    fi
    if [ $i -eq 8 ]; then
        break # Stop at 8
    fi
    echo "Value: $i"
done
```

## Interview Questions

**Q: What is the difference between `while` and `until` loops in Bash?**
**A:** A `while` loop continues executing as long as the condition is true (exit status 0). An `until` loop continues as long as the condition is false (non-zero exit status) and terminates once the condition becomes true.

**Q: How do you read a file line by line safely in a shell script?**
**A:** The safest way is using a `while read` loop: `while IFS= read -r line; do ... done < file.txt`. Using `IFS=` ensures leading/trailing whitespace is preserved, and `-r` prevents backslashes from being treated as escape characters.

**Q: How can you create an infinite loop in Bash?**
**A:** You can use `while true; do ... done` or `for ((;;)); do ... done`. To stop an infinite loop, you usually need to send a signal (like Ctrl+C) or include a `break` statement inside the loop.

**Q: What is the purpose of the `continue` statement?**
**A:** The `continue` statement skips the remaining commands in the current iteration of the loop and moves immediately to the next iteration (re-evaluating the condition for `while`/`until` or taking the next item in `for`).

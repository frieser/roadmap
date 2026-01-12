---
---

## Summary
`while` and `until` are the two primary looping constructs for repeating commands based on a condition. `while` loops as long as the condition is **True**. `until` loops as long as the condition is **False** (until it becomes True).

## Detailed Explanation

### While Loop
```bash
count=0
while [ $count -lt 5 ]; do
    echo $count
    ((count++))
done
```
Often used for reading files (`while read line; do ... done < file`).

### Until Loop
Inverse logic.
```bash
until ping -c 1 8.8.8.8; do
    echo "Waiting for internet..."
    sleep 1
done
```
This runs the ping loop *until* it succeeds (returns 0).

### Infinite Loops
*   `while true; do ... done` (Standard).
*   `while :; do ... done` (Legacy optimization, `:` is a no-op built-in).

## Go-Specific Context/Examples

Go effectively only has `for`.

### Analogy
*   **Bash**: `while [ $a -lt 5 ]`
*   **Go**: `for a < 5 { ... }`

*   **Bash**: `while true`
*   **Go**: `for { ... }`

## Interview Questions

**Q: When should you use `until` instead of `while`?**
**A:** When you are waiting for a specific event to succeed (like a server coming online). `until check_server; do sleep 1; done` reads more naturally than `while ! check_server`.

**Q: How do you break out of an infinite loop?**
**A:** Use the `break` command inside the loop logic.

**Q: What is the most efficient way to read a file line-by-line?**
**A:** `while IFS= read -r line; do ... done < file`. Avoid `for line in $(cat file)` because `cat` loads the whole file into memory and word-splitting breaks lines with spaces.

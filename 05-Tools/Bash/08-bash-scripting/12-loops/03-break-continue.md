---
---

## Summary
`break` and `continue` allow you to control the flow of loops (`for`, `while`, `until`). `break` exits the loop entirely, while `continue` skips the rest of the current iteration and jumps to the next one.

## Detailed Explanation

### Basic Usage
*   **`break`**: Terminates the loop immediately.
*   **`continue`**: Skips remaining commands in the current iteration and starts the next one.

### Nested Loops
You can pass an integer argument to control how many levels to break/continue.
*   `break 2`: Exits the current loop AND the parent loop.
*   `continue 2`: Jumps to the next iteration of the *parent* loop.

### Example
```bash
for i in 1 2 3; do
    for j in a b c; do
        if [ "$i" -eq 2 ] && [ "$j" = "b" ]; then
            break 2  # Breaks BOTH loops
        fi
        echo "$i-$j"
    done
done
```

## Go-Specific Context/Examples

Go supports `break` and `continue`, but uses **Labels** for nested loops instead of numbers.

### Analogy
*   **Bash**: `break 2`
*   **Go**:
    ```go
    OuterLoop:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            break OuterLoop // Explicit label
        }
    }
    ```

## Interview Questions

**Q: What happens if you use `break` outside of a loop?**
**A:** Bash will print a warning: `break: only meaningful in a `for', `while', or `until' loop`. It does not exit the script (unless `set -e` is on depending on version, but usually just a warning).

**Q: Can you use `continue` in a `case` statement?**
**A:** No. `case` is not a loop. Use `;;` to break out of a case. However, if the `case` is *inside* a loop, `continue` inside the `case` will skip to the next loop iteration.

**Q: What is the default value for N in `break N`?**
**A:** 1 (The current loop).

---
---

## Summary
The `case` statement is a control flow structure that allows you to match a variable against several patterns (like a Switch statement). It is generally cleaner than complex `if-elif-else` chains when checking a single variable against multiple values.

## Detailed Explanation

### Syntax
```bash
case "$variable" in
    pattern1)
        command
        ;;
    pattern2|pattern3)
        command
        ;;
    *)
        default_command
        ;;
esac
```

### Terminators
*   **`;;`**: Stop (Break).
*   **`;&`**: Fallthrough (Continue to next block unconditionally).
*   **`;;&`**: Test next pattern (Continue testing).

### Patterns
Patterns support globbing (`*`, `?`, `[a-z]`).
*   `y|Y|yes|Yes)` matches any of those strings.
*   `*.jpg)` matches any string ending in .jpg.

## Go-Specific Context/Examples

Go's `switch` statement is the direct equivalent.

### Analogy
**Bash**:
```bash
case "$1" in
    start) run ;;
    *) echo "Usage: start" ;;
esac
```
**Go**:
```go
switch os.Args[1] {
case "start":
    run()
default:
    fmt.Println("Usage: start")
}
```
**Diff**: Go `switch` breaks by default (no `break` needed). Bash needs explicit `;;`.

## Interview Questions

**Q: How do you match "anything else" (Default case)?**
**A:** Use the wildcards pattern `*)`. It matches everything, so it must be placed at the very end.

**Q: Can you use regex in a case statement?**
**A:** Not directly. `case` uses **Globbing** (wildcards), not Regex. For Regex matching, you need to use `if [[ $var =~ regex ]]`.

**Q: How do you implement case-insensitive matching in a case statement?**
**A:**
1.  Convert var to lower: `case "${var,,}" in ...`.
2.  Use character classes: `[yY][eE][sS])`.
3.  Set `shopt -s nocasematch` before the block.

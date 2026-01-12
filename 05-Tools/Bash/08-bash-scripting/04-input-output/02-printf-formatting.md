---
---

## Summary
`printf` (print formatted) is a robust replacement for `echo`. Unlike `echo`, which has inconsistent behavior across different shells and OSs (handling of `-n`, `-e`), `printf` behaves consistently and allows precise formatting of output (width, alignment, number precision) similar to C's `printf`.

## Detailed Explanation

### Syntax
`printf format-string [arguments...]`

### Format Specifiers
*   `%s`: String.
*   `%d`: Signed integer.
*   `%f`: Floating point number.
*   `%b`: Expand backslash escape sequences in the argument.

### Formatting
*   **Width**: `%10s` (Pad with spaces to 10 chars, right aligned).
*   **Left Align**: `%-10s`.
*   **Precision**: `%.2f` (Float with 2 decimal places).

### Example
```bash
printf "Name: %-10s ID: %04d\n" "Alice" 5
# Output: Name: Alice      ID: 0005
```
*Note: `printf` does NOT add a newline `\n` automatically. You must add it.*

## Go-Specific Context/Examples

Bash `printf` is nearly identical to Go's `fmt.Printf`.

### Analogy
**Bash**:
```bash
printf "User: %s, Score: %d\n" "Bob" 100
```
**Go**:
```go
fmt.Printf("User: %s, Score: %d\n", "Bob", 100)
```

## Interview Questions

**Q: Why use `printf` over `echo`?**
**A:** `echo` is non-portable. `echo -n` works in Bash but might print `-n` literally in `sh` or `ksh`. `printf` is POSIX standard and consistent. Also, `printf` allows formatting (tables, zero-padding) that `echo` cannot do easily.

**Q: How do you assign the output of `printf` to a variable?**
**A:** Use **Command Substitution**.
`my_var=$(printf "%.2f" 3.14159)`

**Q: What happens if you provide more arguments than format specifiers?**
**A:** `printf` reuses the format string until all arguments are consumed.
`printf "%s\n" a b c` outputs:
```
a
b
c
```
This acts like a loop!

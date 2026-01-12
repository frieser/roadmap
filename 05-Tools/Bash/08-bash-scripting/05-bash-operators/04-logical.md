---
---

## Summary
Logical operators allow you to combine multiple test conditions. The syntax depends on whether you are using the old test command `[ ]` or the modern `[[ ]]`.

## Detailed Explanation

### Modern Syntax (`[[ ]]`)
*   `&&`: Logical AND. `[[ condition1 && condition2 ]]`.
*   `||`: Logical OR. `[[ condition1 || condition2 ]]`.
*   `!`: Logical NOT. `[[ ! condition ]]`.

### Legacy Syntax (`[ ]`)
*   `-a`: Logical AND. `[ condition1 -a condition2 ]`. **Deprecated**.
*   `-o`: Logical OR. `[ condition1 -o condition2 ]`. **Deprecated**.
*   It is safer to chain commands: `[ condition1 ] && [ condition2 ]`.

### Short-Circuit Evaluation
Bash uses short-circuit logic.
*   `cmd1 && cmd2`: Run `cmd2` only if `cmd1` succeeds (returns 0).
*   `cmd1 || cmd2`: Run `cmd2` only if `cmd1` fails (returns non-zero).

## Go-Specific Context/Examples

Go uses the standard C-style logical operators `&&`, `||`, `!`.

### Analogy
**Bash**:
```bash
if [[ -f file.txt && -r file.txt ]]; then ...
```
**Go**:
```go
info, err := os.Stat("file.txt")
if err == nil && info.Mode().Perm()&0400 != 0 { ... }
```

## Interview Questions

**Q: Why is `-a` deprecated in `[ ]`?**
**A:** It is ambiguous. If you have `[ "$var" -a -f file ]` and `$var` expands to `"!"`, the command becomes `[ ! -a -f file ]`. The parser gets confused about whether `!` is an operator or a string. `[[ && ]]` handles this correctly.

**Q: What is the difference between `&&` inside `[[ ]]` and `&&` between commands?**
**A:**
*   `[[ A && B ]]`: Boolean logic *inside* the test. Checks if expression A and expression B are true.
*   `cmd1 && cmd2`: Control flow. Executes `cmd2` only if `cmd1` exits successfully.

**Q: How do you negate a condition?**
**A:** `if ! command; then ...`. Or `if [ ! -f file ]; then ...`.

---
---

## Summary
`expr` is a legacy command-line utility for evaluating expressions. It handles integer arithmetic, string matching, and comparison. In modern Bash scripting, it has been largely superseded by Arithmetic Expansion `$((...))` and modern test operators `[[...]]`.

## Detailed Explanation

### Usage
`expr arg1 operator arg2`

### Quirks
*   **Spaces**: Arguments must be separated by spaces. `expr 1+1` prints the string "1+1". `expr 1 + 1` prints "2".
*   **Escaping**: Operators like `*` must be escaped because the shell interprets them as wildcards. `expr 5 \* 5`.

### Comparison
*   `expr 10 = 10` returns 1 (True).
*   `expr 10 \> 5` returns 1.

## Go-Specific Context/Examples

There is no direct Go equivalent because Go handles expressions natively in the language syntax. `expr` is an external binary, making it much slower than shell built-ins.

### Analogy
Running `expr` in a loop is like running `go run math.go` inside a loop instead of just doing `a + b`.

## Interview Questions

**Q: Why is `expr` considered deprecated/legacy?**
**A:** It spawns a new process for every calculation, which is slow. It requires clunky escaping for common symbols (`*`, `(`, `)`). Built-in `$((...))` is faster and cleaner.

**Q: When might you still see `expr`?**
**A:** In very old shell scripts written for strict POSIX `sh` compatibility where `$((...))` might not have been available or standard at the time (though POSIX `sh` does support arithmetic expansion now).

**Q: How do you find the length of a string using `expr`?**
**A:** `expr length "string"`. (In modern Bash: `${#string}`).

---
---

## Summary
The `let` command is a legacy built-in for arithmetic assignment. It works similarly to `((...))`, evaluating integer arithmetic expressions. While still supported, `((...))` is generally preferred for readability.

## Detailed Explanation

### Syntax
`let "variable = expression"`

### Quoting
Unlike `((...))`, `let` requires quoting if the expression contains spaces.
*   `let a=5+5`: Works.
*   `let "a = 5 + 5"`: Works.
*   `let a = 5 + 5`: Fails (syntax error).

### Comparison
*   **Expansion**: `x=$(( 1 + 1 ))` is cleaner than `let "x = 1 + 1"`.
*   **Logic**: `let` returns exit status 1 if the result is 0 (False), and 0 if non-zero (True).

## Go-Specific Context/Examples

Go handles assignment strictly.

### Analogy
*   **Bash**: `let "count++"`
*   **Go**: `count++`

## Interview Questions

**Q: Is `let` faster than `expr`?**
**A:** Yes. `let` is a shell **built-in**, meaning it runs inside the shell process. `expr` is an **external program** (coreutils), requiring a `fork()` and `exec()` call, which is expensive in loops.

**Q: Why prefer `((...))` over `let`?**
**A:** Readability and consistency. `(( a = b + c ))` handles spaces naturally without quotes. Also, `var=$((...))` is the standard way to capture output, whereas `let` only performs side-effect assignment.

**Q: Can you do `let "a = 5.5 + 1"`?**
**A:** No. Like all native Bash arithmetic, `let` is **integer only**.

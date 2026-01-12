---
---

## Summary
Bash supports basic integer arithmetic using the `((...))` expansion or the `let` command. It supports standard operators (`+`, `-`, `*`, `/`, `%`) and even exponentiation (`**`). **Crucial Limitation**: Bash natively only supports **integers**. It cannot handle floating-point numbers (decimals).

## Detailed Explanation

### Syntax
1.  **Expansion**: `$(( 5 + 5 ))` returns the result.
    *   `echo $(( 10 / 2 ))` -> 5.
2.  **Compound Command**: `(( a = 5 + 5 ))`. Used for assignment or logic (returns exit status 0 if non-zero value, 1 if zero).
3.  **let**: `let "a = 5 + 5"`. Older syntax.

### Operators
*   **Arithmetic**: `+`, `-`, `*`, `/` (Integer division), `%` (Modulo), `**` (Power).
*   **Shorthand**: `++`, `--`, `+=`, `*=`.
    *   `(( i++ ))` increments i.

### Floating Point Workaround
To do float math, you must pipe to `bc` (Basic Calculator) or `awk`.
`echo "3.5 + 2.2" | bc` -> 5.7.

## Go-Specific Context/Examples

Go is strictly typed. You cannot mix `int` and `float64` without casting.

### Analogy
*   **Bash**: `echo $(( 10 / 3 ))` -> `3` (Integer division).
*   **Go**: `fmt.Println(10 / 3)` -> `3` (Integer division).
*   **Go (Float)**: `fmt.Println(10.0 / 3.0)` -> `3.333...`.

## Interview Questions

**Q: How do you check if a number is even in Bash?**
**A:** Use the modulo operator `%`.
```bash
if (( num % 2 == 0 )); then echo "Even"; fi
```

**Q: Why does `n=n+1` print `1+1` instead of `2`?**
**A:** Because Bash variables are strings by default. `n=1; n=$n+1` results in the string `1+1`. You must force arithmetic context: `(( n = n + 1 ))` or `let n++`.

**Q: What is the exit status of `(( 0 ))`?**
**A:** **1 (Failure)**. In arithmetic context, a result of 0 is considered "False", so the return code is 1. A non-zero result is "True" (return code 0). This is the inverse of standard exit codes but aligns with C-style boolean logic inside the double parentheses.

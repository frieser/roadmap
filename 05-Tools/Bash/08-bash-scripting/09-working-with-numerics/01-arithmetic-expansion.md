---
---

## Summary
Arithmetic expansion `$((...))` allows for the evaluation of arithmetic expressions directly within the shell. It replaces the expression with the result of the calculation.

## Detailed Explanation

### Syntax
`$(( expression ))`

### Supported Operators
*   **Standard**: `+`, `-`, `*`, `/`, `%` (Remainder).
*   **Exponentiation**: `**` (e.g., `2**3` is 8).
*   **Increment/Decrement**: `var++`, `--var`.
*   **Bitwise**: `<<`, `>>`, `&`, `|`, `^`.
*   **Ternary**: `cond ? true_val : false_val`.

### Variables
Inside `$((...))`, you don't need the `$` prefix for variables.
`a=5; echo $(( a + 5 ))` -> 10.

## Go-Specific Context/Examples

Go syntax is stricter but similar for basic math.

### Analogy
**Bash**: `res=$(( (a + b) * c ))`
**Go**: `res := (a + b) * c`

Go does **not** support `**` for power (must use `math.Pow`), whereas Bash does.

## Interview Questions

**Q: Does `$((...))` support floating point numbers?**
**A:** No. Bash arithmetic is strictly **integer-only**. `$(( 5 / 2 ))` equals `2`, not `2.5`. For floats, use `bc`.

**Q: How do you generate a random number between 0 and 9?**
**A:** `echo $(( RANDOM % 10 ))`. `$RANDOM` is a special Bash variable returning an integer between 0 and 32767.

**Q: What is the result of `echo $(( 2#101 ))`?**
**A:** **5**. This syntax `base#number` allows converting numbers from different bases (binary 101 -> decimal 5).

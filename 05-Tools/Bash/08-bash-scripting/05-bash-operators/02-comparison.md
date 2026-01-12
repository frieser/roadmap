---
---

## Summary
Comparison operators in Bash are a frequent source of confusion because the syntax differs depending on whether you are comparing **Integers** or **Strings**, and whether you use `[ ]` (test) or `[[ ]]` (new test).

## Detailed Explanation

### Integer Operators
Use these inside `[ ... ]` or `[[ ... ]]`.
*   `-eq`: Equal.
*   `-ne`: Not Equal.
*   `-gt`: Greater Than.
*   `-lt`: Less Than.
*   `-ge`: Greater or Equal.
*   `-le`: Less or Equal.

### String Operators
*   `=`: Equal (POSIX). `==` works in Bash `[[ ... ]]`.
*   `!=`: Not Equal.
*   `-z`: String is empty (Zero length).
*   `-n`: String is not empty.

### `[ ]` vs `[[ ]]`
*   **`[ ]`**: POSIX standard. Variables must be quoted `"$VAR"` to avoid splitting. `>` and `<` must be escaped `\>`.
*   **`[[ ]]`**: Bash extension. Safer. No quoting needed for variables. Supports pattern matching (`== value*`) and regex (`=~`). Supports `&&` and `||`.

## Go-Specific Context/Examples

Go avoids this confusion by using standard operators (`==`, `>`, `<`) for everything and strictly enforcing types at compile time. You can't compare int and string.

### Analogy
*   **Bash**: `if [ "$a" -eq "$b" ]` (Int) vs `if [ "$a" = "$b" ]` (String).
*   **Go**: `if a == b` (Works for both, if types match).

## Interview Questions

**Q: What happens if you use `-eq` to compare strings?**
**A:** Bash tries to interpret the strings as numbers. If they are non-numeric strings (like "abc"), they evaluate to 0. So `[ "abc" -eq "def" ]` is effectively `[ 0 -eq 0 ]`, which returns **True**. This is a dangerous bug. Always use `=` for strings.

**Q: Why prefer `[[ ]]` over `[ ]`?**
**A:** `[[ ]]` is safer and more powerful. It prevents word splitting issues (so `[ -z $VAR ]` won't crash if VAR has spaces), supports logical operators `&&`/`||` directly inside, and allows glob matching (`[[ $VAR == f* ]]`).

**Q: How do you compare floating point numbers?**
**A:** Bash cannot do it directly. You must use `bc` or `awk`.
`if (( $(echo "$a > $b" | bc -l) )); then ...`

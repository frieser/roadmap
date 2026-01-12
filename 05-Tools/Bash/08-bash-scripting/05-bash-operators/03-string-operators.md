---
---

## Summary
String operators in Bash allow you to compare strings for equality, inequality, and lexicographical order. Unlike integer comparisons (`-eq`), string comparisons use symbols like `=`, `!=`, `<`, and `>`. Proper quoting is essential to handle empty strings or spaces safely.

## Detailed Explanation

### Equality
*   `[ "$a" = "$b" ]`: True if equal (POSIX standard).
*   `[[ "$a" == "$b" ]]`: True if equal (Bash extension, supports pattern matching).
*   `[ "$a" != "$b" ]`: True if not equal.

### Ordering
*   `[[ "$a" < "$b" ]]`: True if $a sorts before $b (ASCII order).
*   `[[ "$a" > "$b" ]]`: True if $a sorts after $b.
*   **Note**: Inside `[ ]`, `<` and `>` must be escaped (`\<`, `\>`) or they are treated as redirection operators. Inside `[[ ]]`, no escaping is needed.

### Length Checks
*   `-z "$var"`: True if string is **Zero** length (Empty).
*   `-n "$var"`: True if string is **Non-zero** length (Not empty).

## Go-Specific Context/Examples

In Go, string comparison is built-in with operators, but for ordering/sorting, you typically use the `strings` package.

### Analogy
*   **Bash**: `if [ "$a" = "$b" ]`
*   **Go**: `if a == b`

*   **Bash**: `if [[ "$a" < "$b" ]]`
*   **Go**: `if a < b` (Lexicographical) or `strings.Compare(a, b) == -1`

## Interview Questions

**Q: Why does `[ $var = "foo" ]` fail if `$var` is empty?**
**A:** If `$var` is empty and unquoted, the command becomes `[ = "foo" ]`, which is a syntax error (unary operator expected). Quoting it `[ "$var" = "foo" ]` expands to `[ "" = "foo" ]`, which is valid.

**Q: What is the difference between `-z` and `== ""`?**
**A:** Functionally, they both check for emptiness. `[ -z "$var" ]` is the idiomatic standard. `[ "$var" == "" ]` works but is slightly more verbose.

**Q: How do you perform case-insensitive comparison?**
**A:** 
1.  Convert to lowercase: `if [ "${var,,}" = "value" ]` (Bash 4.0+).
2.  Use `shopt -s nocasematch` inside `[[ ]]`.
3.  Use `grep -i`.

---
---

## Summary
In Bash 4.0+, you can easily convert the case of strings using the parameter expansion syntax `${var^^}` (Uppercase) and `${var,,}` (Lowercase). This eliminates the need for `tr` or `awk` for simple case changes.

## Detailed Explanation

### Syntax
*   `${var^^}`: Uppercase **all** letters.
*   `${var,,}`: Lowercase **all** letters.
*   `${var^}`: Uppercase **first** letter (Capitalize).
*   `${var,}`: Lowercase **first** letter.
*   `${var^^[aeiou]}`: Uppercase only specific characters (e.g., vowels).

### Examples
`v="hello world"`
*   `${v^^}` -> `HELLO WORLD`
*   `${v^}` -> `Hello world`

## Go-Specific Context/Examples

In Go, the `strings` package handles unicode-aware casing.

### Analogy
*   **Bash**: `${s^^}`
*   **Go**: `strings.ToUpper(s)`

*   **Bash**: `${s^}` (Title case first letter)
*   **Go**: `cases.Title(language.English).String(s)` (Go's `strings.Title` is deprecated).

## Interview Questions

**Q: How do you do case conversion in older Bash (<4.0) or `sh`?**
**A:** You must pipe to `tr`.
`echo "$var" | tr '[:lower:]' '[:upper:]'`

**Q: Does `${var^^}` change the variable itself?**
**A:** No, it returns a modified copy (expansion). To change the variable, you must reassign it: `var=${var^^}`. However, you can declare a variable as explicitly "Upper Attribute" using `declare -u var`. Any value assigned to it is auto-uppercased.

**Q: Does it handle UTF-8?**
**A:** Yes, in modern Bash with proper locale settings, it handles characters like `ñ` -> `Ñ`.

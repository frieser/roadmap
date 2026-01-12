---
---

## Summary
Getting the length of a string in Bash is done using the parameter expansion syntax `${#VAR}`. This is fast, built-in, and avoids piping to `wc`.

## Detailed Explanation

### Syntax
*   `${#variable}`: Returns the length of the value in characters.

### Examples
```bash
str="hello"
echo ${#str}  # Output: 5
```

### Arrays vs Strings
*   `${#arr}`: Length of the *first element* (index 0).
*   `${#arr[@]}`: Number of *elements* in the array.

## Go-Specific Context/Examples

In Go, `len(s)` returns **bytes**, not characters (runes).

### Analogy
*   **Bash**: `${#s}` counts characters (multibyte aware in modern Bash).
*   **Go**: `len("ñ")` is 2 (bytes). `utf8.RuneCountInString("ñ")` is 1 (character).

## Interview Questions

**Q: How do you check if a string length is zero?**
**A:** `if [ -z "$str" ]; then ...` (Preferred). Using `if [ ${#str} -eq 0 ];` also works but is more verbose.

**Q: What is the overhead of `echo "$str" | wc -c` vs `${#str}`?**
**A:** Piping to `wc` forks a new process (`subshell` + `wc`). `${#str}` is a shell built-in parameter expansion, occurring instantly in memory. In a tight loop, the built-in is orders of magnitude faster.

**Q: Does `${#var}` count bytes or characters?**
**A:** It counts **characters** based on the current locale (UTF-8 safe).

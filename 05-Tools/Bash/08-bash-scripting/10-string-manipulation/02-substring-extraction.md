---
---

## Summary
Bash provides parameter expansion syntax to extract substrings from a variable based on offset and length. The format is `${var:offset:length}`.

## Detailed Explanation

### Syntax
`${parameter:offset:length}`

### Examples
`str="0123456789"`
*   **Offset only**: `${str:7}` -> "789" (Start at index 7 to end).
*   **Offset & Length**: `${str:0:3}` -> "012" (Start at 0, take 3).
*   **Negative Offset**: `${str: -3}` -> "789" (Take last 3). **Note the space** before `-3`. Without space, it conflicts with default value syntax `${var:-default}`.

## Go-Specific Context/Examples

Go supports slicing strings (which are byte slices) natively.

### Analogy
*   **Bash**: `${str:1:4}` (from index 1, take 4 chars).
*   **Go**: `str[1:5]` (from index 1, up to *but not including* 5).
    *   *Warning*: Go slicing operates on bytes. Slicing a multibyte string in Go might split a rune in half. Bash handles characters safely.

## Interview Questions

**Q: Why do you need a space for negative offset `${var: -1}`?**
**A:** Because `${var:-1}` is already defined as "Return `$var` if set, otherwise return literal `1`" (Default Value expansion). The space disambiguates it to mean "Offset -1".

**Q: How to get the last character of a string?**
**A:** `${var: -1}`.

**Q: What happens if length exceeds the string end?**
**A:** Bash returns characters up to the end of the string. It does not throw an "Index Out of Bounds" error like Python or Go.

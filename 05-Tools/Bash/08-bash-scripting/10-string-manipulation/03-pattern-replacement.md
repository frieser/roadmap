---
---

## Summary
Bash allows replacing text within a variable using parameter expansion `${var/pattern/replacement}`. This acts like a lightweight `sed` operating directly on variables.

## Detailed Explanation

### Syntax
*   **Replace First**: `${var/pattern/replace}`
*   **Replace All**: `${var//pattern/replace}` (Double slash).
*   **Delete**: `${var/pattern/}` (Replace with nothing).
*   **Anchor Start**: `${var/#pattern/replace}` (Only if it starts with pattern).
*   **Anchor End**: `${var/%pattern/replace}` (Only if it ends with pattern).

### Examples
`path="/home/user/docs/file.txt"`
*   `${path/user/admin}` -> `/home/admin/docs/file.txt`
*   `${path//\//-}` -> `-home-user-docs-file.txt` (Replace all slashes with dashes).

## Go-Specific Context/Examples

In Go, `strings` package functions are the direct equivalent.

### Analogy
*   **Bash**: `${s//old/new}`
*   **Go**: `strings.ReplaceAll(s, "old", "new")`

*   **Bash**: `${s/old/new}`
*   **Go**: `strings.Replace(s, "old", "new", 1)`

## Interview Questions

**Q: Can you use wildcards in the pattern?**
**A:** Yes. `${var/*.txt/text_file}` matches the pattern using standard globbing rules (`*`, `?`, `[]`), not full regex.

**Q: How do you delete a specific substring?**
**A:** Omit the replacement part. `${var/substring/}`.

**Q: How to change the extension of a file path?**
**A:** Use the "End Anchor" `%`.
`file="image.png"` -> `${file/%.png/.jpg}` -> `image.jpg`.

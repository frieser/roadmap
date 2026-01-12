---
---

## Summary
Regular Expressions (Regex) are patterns used to match character combinations in strings. In Bash tools (`grep`, `sed`), there are two main flavors: **Basic Regular Expressions (BRE)** and **Extended Regular Expressions (ERE)**. Understanding the difference is key to avoiding syntax errors.

## Detailed Explanation

### Anchors
*   `^`: Start of line.
*   `$`: End of line.

### Character Classes
*   `.`: Any single character.
*   `[abc]`: Any one of a, b, or c.
*   `[a-z]`: Range.
*   `[^a-z]`: Negated range (Not a-z).

### Quantifiers (The Confusion Point)
*   **ERE (`grep -E`, `sed -r`)**: `?` (0 or 1), `+` (1 or more), `{n,m}` work directly. `|` (OR) works directly.
*   **BRE (Default `grep`, `sed`)**: To use `?`, `+`, `{`, `|`, `(`, `)`, you MUST **escape** them: `\?`, `\+`, `\{`, `\|`, `\(`, `\)`.
    *   *Wait, what?* Yes. In BRE, `(` matches a literal paren. `\(` starts a capture group. In ERE, `(` starts a capture group.

## Go-Specific Context/Examples

Go's `regexp` package uses **RE2** syntax, which is mostly compatible with ERE (Extended) and Perl-like syntax (`\d`, `\s`). It specifically avoids features that cause exponential backtracking (like backreferences).

### Analogy
*   **Bash (BRE)**: `grep "a\+"`
*   **Go (RE2)**: `regexp.MustCompile("a+")`

## Interview Questions

**Q: Why doesn't `grep "a+b" file` work?**
**A:** By default, `grep` uses BRE. In BRE, `+` matches a literal plus sign. To make it a quantifier (one or more), you must escape it: `grep "a\+b"` OR switch to ERE: `grep -E "a+b"`.

**Q: What is the difference between `*` in bash globbing vs regex?**
**A:**
*   **Glob (`ls *.txt`)**: Matches any string of characters (filename).
*   **Regex (`grep "a*"`)**: Quantifier. Matches **zero or more** of the *preceding element* ("a").

**Q: How do you match a literal `.` (dot)?**
**A:** Escape it: `\.` or put it in a class `[.]`.

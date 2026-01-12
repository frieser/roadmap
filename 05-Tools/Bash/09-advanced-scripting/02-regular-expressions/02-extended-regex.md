---
---

## Summary
Extended Regular Expressions (ERE) simplify regex syntax by removing the need to escape meta-characters like `+`, `?`, `|`, `(`, and `)`. This makes regexes more readable and consistent with modern languages. You enable ERE using `grep -E` (or `egrep`) and `sed -r` (GNU) or `sed -E` (BSD/macOS).

## Detailed Explanation

### Syntax Differences (BRE vs ERE)
| Feature | BRE (Basic) | ERE (Extended) |
| :--- | :--- | :--- |
| One or more | `\+` | `+` |
| Zero or one | `\?` | `?` |
| Alternation (OR) | `\|` | `|` |
| Grouping | `\( \)` | `( )` |
| Interval | `\{n,m\}` | `{n,m}` |

### Examples
*   **BRE**: `grep "http\|https"` (Escaped pipe).
*   **ERE**: `grep -E "http|https"` (Clean pipe).
*   **BRE**: `grep "\(abc\)\+"` (Escaped parens and plus).
*   **ERE**: `grep -E "(abc)+"` (Clean).

## Go-Specific Context/Examples

Go's `regexp` package supports RE2, which is syntactically almost identical to ERE.

### Analogy
*   **Bash (grep -E)**: `[a-z]+`
*   **Go**: `regexp.MustCompile("[a-z]+")`

## Interview Questions

**Q: Why do some systems alias `egrep` to `grep -E`?**
**A:** `egrep` is deprecated. The POSIX standard defines `grep -E` as the way to invoke Extended Regex. Aliasing ensures backward compatibility for older scripts.

**Q: What is PCRE?**
**A:** Perl Compatible Regular Expressions. It is even more powerful than ERE (supports lookaheads, lookbehinds, `\d`, `\w`). Use `grep -P` to enable it (if available). Go's `regexp` does NOT support PCRE (no lookaheads) to guarantee O(n) execution time.

**Q: How do you match a literal `+` in ERE?**
**A:** Escape it: `\+`. (In BRE, `+` is literal by default).

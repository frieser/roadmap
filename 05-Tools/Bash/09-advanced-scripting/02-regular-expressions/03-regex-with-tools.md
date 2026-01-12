---
---

## Summary
The "Holy Trinity" of text processing tools in Unix are **grep**, **sed**, and **awk**. Each uses regex differently.
*   **grep**: Search (Filter lines).
*   **sed**: Edit (Search and Replace).
*   **awk**: Analyze (Columnar data processing).

## Detailed Explanation

### grep (Global Regular Expression Print)
*   `grep "pattern" file` (BRE).
*   `grep -E "pattern" file` (ERE).
*   `grep -o "pattern"` (Output only the matched segment).

### sed (Stream Editor)
*   `sed 's/pattern/replacement/g' file`.
*   Uses BRE by default. Use `-E` (or `-r`) for ERE.
*   `&` represents the matched string in replacement.

### awk
*   `awk '/pattern/ { action }' file`.
*   Uses ERE by default.
*   `awk '$1 ~ /^[0-9]+$/ { print $2 }'` (If column 1 matches regex digits, print column 2).

## Go-Specific Context/Examples

In Go, you typically import `regexp` to do what all three tools do.

### Analogy: grep
**Bash**: `grep "err" file.log`
**Go**:
```go
if strings.Contains(line, "err") { fmt.Println(line) }
// or regex.MatchString
```

### Analogy: sed
**Bash**: `sed 's/foo/bar/g'`
**Go**: `regexp.MustCompile("foo").ReplaceAllString(input, "bar")`

## Interview Questions

**Q: Which tool is best for extracting the 3rd IP address from a line?**
**A:** **awk**. `awk` splits fields by whitespace automatically. `awk '{print $3}'`. Using regex with `grep` or `sed` for positional field extraction is painful.

**Q: How do you use regex groups in `sed` replacement?**
**A:** Use `\1`, `\2`, etc.
`echo "Item: 123" | sed -E 's/Item: ([0-9]+)/ID=\1/'` -> `ID=123`.

**Q: Can `grep` replace text?**
**A:** No. `grep` only searches and prints. Use `sed` for replacement.

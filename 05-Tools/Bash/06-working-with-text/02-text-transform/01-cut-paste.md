---
---

## Summary
`cut` and `paste` are complementary tools for processing text columns. `cut` extracts vertical columns (fields) from text, while `paste` merges files horizontally by combining lines.

## Detailed Explanation

### Cut (Extract Columns)
Used to parse delimited data (CSV, logs, `/etc/passwd`).
*   **Delimiter**: `-d ':'` (Default is Tab).
*   **Fields**: `-f 1,3` (Columns 1 and 3). `-f 2-` (Column 2 to end).
*   **Chars**: `-c 1-5` (First 5 characters).

**Example**: Get usernames from `/etc/passwd`
`cut -d ':' -f 1 /etc/passwd`

### Paste (Merge Columns)
Combines lines from file A and file B side-by-side.
*   `paste file1.txt file2.txt`: Output `line1_A <tab> line1_B`.
*   `-d ','`: Use comma delimiter (Creating CSV).
*   `-s`: Serial (Paste one file at a time horizontally).

## Go-Specific Context/Examples

In Go, string manipulation is handled by the `strings` package or `encoding/csv`.

### Analogy: Cut
**Bash**: `cut -d ',' -f 2`
**Go**:
```go
parts := strings.Split(line, ",")
if len(parts) >= 2 {
    fmt.Println(parts[1])
}
```

### Analogy: Paste
**Bash**: `paste -d ',' a.txt b.txt`
**Go**: Reading two files line-by-line and formatting output.

## Interview Questions

**Q: Can `cut` reorder columns?**
**A:** No. `cut -f 3,1` will output column 1 then column 3 (in the original order). `cut` only extracts; it doesn't rearrange. To reorder, use `awk '{print $3, $1}'`.

**Q: How do you extract the last column if you don't know how many there are?**
**A:** `cut` generally needs fixed field numbers. To get the "last" field dynamically, `awk` is better (`awk '{print $NF}'`) or `rev | cut -f 1 | rev` (Reverse line, cut first, reverse back).

**Q: What happens if `paste` encounters files of different lengths?**
**A:** It fills the missing lines from the shorter file with empty strings (just the delimiter), keeping the structure aligned.

---
---

## Summary
`join` and `split` are powerful text manipulation utilities. **`join`** performs a relational join on two files based on a common field (like an SQL `INNER JOIN`). **`split`** breaks a large file into smaller chunks based on size or line count.

## Detailed Explanation

### Join (Relational Merge)
Merges lines from two files that share a join field. **Crucial**: Input files MUST be sorted on the join field.
*   **Syntax**: `join [options] file1 file2`
*   **Fields**: `-1 1 -2 1` (Join on field 1 of file 1 and field 1 of file 2).
*   **Output**: Only lines with matching join fields are printed (Inner Join).
*   **Unmatched**: `-a 1` (Left Join - print unmatched lines from file 1).

### Split (Chunking)
Breaks a file into pieces (e.g., `xaa`, `xab`, `xac`).
*   **By Lines**: `split -l 1000 bigfile.txt` (1000 lines per file).
*   **By Bytes**: `split -b 100M bigfile.txt` (100MB per file).
*   **Prefix**: `split -l 500 log.txt log_part_` (Output `log_part_aa`).

## Go-Specific Context/Examples

### Analogy: Join
**Bash**: `join file1.txt file2.txt`
**Go**: Implementing a Hash Join or Merge Join algorithm.
```go
// Map-based join (Hash Join)
map1 := make(map[string]string)
// ... load file1 into map ...
for key, val2 := range file2Data {
    if val1, ok := map1[key]; ok {
        fmt.Println(key, val1, val2)
    }
}
```

### Analogy: Split
**Bash**: `split -b 10M`
**Go**: Reading a stream and creating a new file every N bytes.

## Interview Questions

**Q: Why is `join` failing or missing matches?**
**A:** The most common reason is that the input files are **not sorted**. `join` processes files linearly; if line 5 has key "Z" and line 6 has key "A", it won't backtrack. Always run `sort file1 > file1.sorted` before joining.

**Q: How do you reassemble files split by `split`?**
**A:** Use `cat`. `cat xaa xab xac > original_file.txt`.

**Q: How does `split` name the output files?**
**A:** By default, it uses a suffix of `aa`, `ab`, `ac`, etc. You can change this to numeric suffixes with `-d` (`x00`, `x01`) and change the suffix length with `-a`.

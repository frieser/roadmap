---
---

## Summary
The `sort` command arranges the lines of a text file in a specified order. It is one of the most frequently used tools in data processing pipelines, often preceding commands like `uniq` or `join`.

## Detailed Explanation

### Common Options
*   **Default**: Sorts alphabetically (ASCII order). `10` comes before `2`.
*   **`-n` (Numeric)**: Sorts by arithmetic value. `2` comes before `10`.
*   **`-r` (Reverse)**: Descending order.
*   **`-u` (Unique)**: Removes duplicate lines (like `sort | uniq`).
*   **`-k` (Key)**: Sort by a specific column. `sort -k 2` (Sort by 2nd field).
*   **`-t` (Separator)**: Define delimiter. `sort -t ',' -k 3` (Sort CSV by 3rd column).

### Human Readable
*   **`-h`**: Sorts sizes like `2K`, `1G`, `500M` correctly.

## Go-Specific Context/Examples

Go's standard library `sort` package provides interfaces for sorting.

### Analogy
**Bash**: `sort -n numbers.txt`
**Go**:
```go
import "sort"

ints := []int{10, 2, 5}
sort.Ints(ints) // [2, 5, 10]

strs := []string{"b", "a", "c"}
sort.Strings(strs) // ["a", "b", "c"]
```

### Custom Sort (Like `-k`)
```go
type Person struct { Name string; Age int }
// Sort by Age
sort.Slice(people, func(i, j int) bool {
    return people[i].Age < people[j].Age
})
```

## Interview Questions

**Q: Is `sort` stable?**
**A:** By default, GNU `sort` is stable (if two lines compare equal, their relative order remains unchanged). However, this depends on the implementation (`--stable` flag guarantees it).

**Q: How do you sort by the second column numerically?**
**A:** `sort -k 2n`.

**Q: Why does `sort` behave differently on different machines?**
**A:** **Locale (`LC_ALL`)**. Some locales ignore case or handle special characters differently. For strict byte-value sorting (consistent across machines), run `LC_ALL=C sort`.

---
---

## Summary
`uniq` is a utility for filtering or reporting repeated lines in a file. **Critical Constraint**: `uniq` only detects *adjacent* duplicate lines. Therefore, it is almost always used after sorting the input (e.g., `sort file.txt | uniq`).

## Detailed Explanation

### Common Options
*   **Default**: Prints unique lines (removes adjacent duplicates).
*   **`-c` (Count)**: Prefix lines by the number of occurrences.
*   **`-d` (Repeated)**: Only print duplicate lines (one for each group).
*   **`-u` (Unique)**: Only print unique lines (discard anything that appears more than once).
*   **`-i`**: Ignore case.

### Usage Pattern
`sort access.log | uniq -c | sort -nr`
1.  **sort**: Group identical lines together.
2.  **uniq -c**: Count occurrences.
3.  **sort -nr**: Sort by count (Numeric, Reverse) to see top hits.

## Go-Specific Context/Examples

Go does not have a built-in "Set" or "Uniq" function for slices, so you implement it manually.

### Example: Deduplication Logic in Go
```go
package main

import "fmt"

func unique(slice []string) []string {
	keys := make(map[string]bool)
	list := []string{}
	for _, entry := range slice {
		if _, value := keys[entry]; !value {
			keys[entry] = true
			list = append(list, entry)
		}
	}
	return list
}

func main() {
	data := []string{"a", "b", "a", "c", "b"}
	fmt.Println(unique(data)) // [a b c]
}
```

## Interview Questions

**Q: Why does `uniq` fail to remove duplicates in an unsorted file?**
**A:** `uniq` only compares the current line with the previous line (buffer size = 1 line). If "apple" appears on line 1 and line 50, `uniq` won't see them as duplicates unless they are next to each other.

**Q: How do you find lines that appear exactly once?**
**A:** `sort file.txt | uniq -u`.

**Q: How do you find the most frequent IP address in a log file?**
**A:** `awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 1`.

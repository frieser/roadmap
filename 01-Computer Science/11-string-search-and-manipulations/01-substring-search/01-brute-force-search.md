---
---

## Summary
**Brute Force Substring Search** is the simplest algorithm for finding a pattern within a text. It works by checking the pattern at every possible position in the text. While easy to implement, it is inefficient for large texts.

## Detailed Explanation

The algorithm slides the pattern over the text one character at a time. At each position, it checks if the characters of the pattern match the corresponding characters in the text.

### Mechanism
1.  Align the pattern with the beginning of the text.
2.  Compare characters from left to right.
3.  If a mismatch occurs, shift the pattern **one position** to the right.
4.  Reset the comparison to the first character of the pattern.
5.  Repeat until a match is found or the end of the text is reached.

### Complexity
*   **Time Complexity**: $O(N \times M)$ in the worst case (e.g., Text="AAAAA...", Pattern="AAAB").
*   **Space Complexity**: $O(1)$ (no extra space needed).

## Go Example

```go
package main

import "fmt"

func BruteForceSearch(text, pattern string) int {
	n := len(text)
	m := len(pattern)

	if m == 0 {
		return 0
	}
	if m > n {
		return -1
	}

	// Iterate through every possible starting position
	for i := 0; i <= n-m; i++ {
		match := true
		// Check the pattern match at this position
		for j := 0; j < m; j++ {
			if text[i+j] != pattern[j] {
				match = false
				break
			}
		}
		if match {
			return i // Found at index i
		}
	}

	return -1 // Not found
}

func main() {
	text := "Hello, World!"
	pattern := "World"
	index := BruteForceSearch(text, pattern)
	fmt.Printf("Found '%s' at index: %d\n", pattern, index)
}
```

## Interview Questions

### Q: When is Brute Force Search acceptable to use?
**A:** It is acceptable for small inputs, one-off scripts, or when the overhead of complex pre-processing (like in KMP or Boyer-Moore) outweighs the search time. It's also useful as a baseline implementation for testing optimized algorithms.

### Q: What is the worst-case scenario for Brute Force Search?
**A:** The worst case occurs when the text and pattern consist of the same repeating character, except for the last character of the pattern (e.g., Text: "AAAAAAAAZB", Pattern: "AAAAAB"). The algorithm compares `M` characters for every `N` positions.

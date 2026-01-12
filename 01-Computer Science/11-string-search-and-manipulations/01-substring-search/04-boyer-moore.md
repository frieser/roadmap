---
---

## Summary
The **Boyer-Moore Algorithm** is considered one of the most efficient string search algorithms in practice. It scans the pattern from **right to left** (opposite to standard search) and uses two powerful heuristics—**Bad Character Rule** and **Good Suffix Rule**—to skip large sections of the text.

## Detailed Explanation

Boyer-Moore relies on the observation that if the last character of the pattern doesn't match the current character in the text, and that text character doesn't appear anywhere in the pattern, we can shift the entire pattern past that character immediately.

### Heuristics
1.  **Bad Character Rule**:
    *   Upon mismatch, shift the pattern so that the mismatched character in the text aligns with the rightmost occurrence of that character in the pattern.
    *   If the character doesn't exist in the pattern, shift the pattern completely past the mismatch.

2.  **Good Suffix Rule** (Complex, often omitted in simplified "Boyer-Moore-Horspool"):
    *   If a suffix of the pattern matches the text but a mismatch occurs earlier, shift the pattern to align another occurrence of that suffix (or a prefix of the pattern that matches the suffix).

### Complexity
*   **Best Case**: $O(N/M)$ (Sub-linear! It doesn't inspect every character).
*   **Worst Case**: $O(N \times M)$ (rare, usually $O(N+M)$).
*   **Space**: $O(\Sigma)$ for the bad character table (where $\Sigma$ is alphabet size).

## Go Example (Boyer-Moore-Horspool)

This example implements the **Horspool** variant, which simplifies Boyer-Moore by only using the Bad Character Rule. It is nearly as fast and much easier to implement.

```go
package main

import "fmt"

func BoyerMooreHorspool(text, pattern string) int {
	n := len(text)
	m := len(pattern)
	if m == 0 {
		return 0
	}
	if m > n {
		return -1
	}

	// 1. Preprocessing: Bad Character Table
	// Stores the distance to shift for each character in the alphabet
	table := make(map[byte]int)
	
	// Default shift is the length of the pattern
	for i := 0; i < 256; i++ {
		// Assuming ASCII. For full unicode, consider map[rune]int
	}

	// For characters in pattern, distance is distance from the end (m - 1 - index)
	// We skip the last character because we want the shift for the *next* occurrence
	for i := 0; i < m-1; i++ {
		table[pattern[i]] = m - 1 - i
	}

	// 2. Searching
	i := m - 1 // Align with end of pattern
	for i < n {
		k := 0
		// Scan right-to-left
		for k < m && pattern[m-1-k] == text[i-k] {
			k++
		}

		if k == m {
			return i - m + 1 // Found match
		}

		// Shift based on the character in TEXT at the current alignment end (text[i])
		// Not the mismatch character, but the one aligning with the end of pattern
		shift, exists := table[text[i]]
		if !exists {
			shift = m
		}
		i += shift
	}

	return -1
}

func main() {
	text := "THIS IS A SIMPLE EXAMPLE"
	pattern := "EXAMPLE"
	fmt.Println("Found at index:", BoyerMooreHorspool(text, pattern))
}
```

## Interview Questions

### Q: Why does Boyer-Moore scan from right to left?
**A:** Scanning from right to left allows the algorithm to maximize the shift distance. If the last character of the pattern mismatches a character in the text that doesn't appear in the pattern at all, we can shift the entire pattern length `M` instantly.

### Q: What is the difference between Boyer-Moore and Horspool?
**A:** Boyer-Moore uses both the **Bad Character** and **Good Suffix** rules. Horspool is a simplification that uses **only** the Bad Character rule. Horspool is simpler to implement and often performs comparably in practice for large alphabets (like ASCII/Unicode).

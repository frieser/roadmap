---
---

## Summary
The **Knuth-Morris-Pratt (KMP)** algorithm is an efficient string searching algorithm that achieves **$O(N+M)$** time complexity. It avoids redundant comparisons by using information about previous matches. It relies on a precomputed **LPS (Longest Prefix Suffix)** array (also called the $\pi$ table).

## Detailed Explanation

When a mismatch occurs, brute force shifts the pattern by 1. KMP uses the LPS array to determine the maximum number of characters we can skip, jumping ahead to the next potential match position without re-evaluating known characters.

### The LPS Array
*   **LPS[i]** stores the length of the longest **proper prefix** of `pattern[0...i]` that is also a **suffix** of `pattern[0...i]`.
*   Example: Pattern "ABABAC"
    *   "A" -> 0
    *   "AB" -> 0
    *   "ABA" -> 1 ("A")
    *   "ABAB" -> 2 ("AB")
    *   "ABABA" -> 3 ("ABA")
    *   "ABABAC" -> 0

### Mechanism
1.  Precompute the LPS array for the pattern.
2.  Traverse the text.
3.  If characters match, advance both text and pattern pointers.
4.  If mismatch occurs at pattern index `j`:
    *   Do NOT move the text pointer back.
    *   Move the pattern pointer `j` to `LPS[j-1]`.
    *   This aligns the prefix of the pattern with the matching suffix found so far.

## Go Example

```go
package main

import "fmt"

// Build the Longest Prefix Suffix (LPS) array
func computeLPS(pattern string) []int {
	m := len(pattern)
	lps := make([]int, m)
	length := 0 // Length of previous longest prefix suffix
	i := 1

	for i < m {
		if pattern[i] == pattern[length] {
			length++
			lps[i] = length
			i++
		} else {
			if length != 0 {
				length = lps[length-1]
			} else {
				lps[i] = 0
				i++
			}
		}
	}
	return lps
}

func KMPSearch(text, pattern string) int {
	n := len(text)
	m := len(pattern)
	if m == 0 {
		return 0
	}

	lps := computeLPS(pattern)
	i := 0 // index for text
	j := 0 // index for pattern

	for i < n {
		if pattern[j] == text[i] {
			j++
			i++
		}
		if j == m {
			return i - j // Found match
			// j = lps[j-1] // To find next match
		} else if i < n && pattern[j] != text[i] {
			if j != 0 {
				j = lps[j-1]
			} else {
				i++
			}
		}
	}
	return -1
}

func main() {
	text := "ABABDABACDABABCABAB"
	pattern := "ABABCABAB"
	fmt.Println("Found at index:", KMPSearch(text, pattern))
}
```

## Interview Questions

### Q: Why is KMP better than Brute Force?
**A:** KMP guarantees linear time complexity $O(N)$ in the worst case, whereas Brute Force can degrade to $O(N \times M)$. KMP never backtracks the text pointer, making it suitable for processing streams of data.

### Q: What does the value `LPS[i]` represent?
**A:** It represents the length of the longest proper prefix of the sub-pattern `P[0...i]` that is also a suffix of that sub-pattern. This value tells us how much of the pattern we have "already matched" internally, allowing us to skip comparisons.

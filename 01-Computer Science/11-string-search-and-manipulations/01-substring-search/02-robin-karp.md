---
---

## Summary
The **Rabin-Karp Algorithm** uses **hashing** to find a pattern in a text. Instead of comparing characters individually at every step, it compares the **hash value** of the pattern with the hash value of the current substring window. It uses a **Rolling Hash** technique to update the hash value in constant time.

## Detailed Explanation

The core idea is that if two strings are equal, their hash values must be equal. However, if hash values are equal, the strings *might* be equal (collision).

### Mechanism
1.  Calculate the hash of the `pattern`.
2.  Calculate the hash of the first substring of `text` (length `M`).
3.  Slide the window one character to the right:
    *   **Rolling Hash**: Remove the leading character's value and add the new trailing character's value to update the hash in $O(1)$ time.
4.  If the hash values match, perform a character-by-character check to confirm (handling collisions).

### Complexity
*   **Average Case**: $O(N + M)$
*   **Worst Case**: $O(N \times M)$ (if many hash collisions occur, effectively degrading to brute force).
*   **Space Complexity**: $O(1)$.

## Go Example

```go
package main

import "fmt"

const (
	// Prime number for modulo operator to reduce collisions
	Prime = 101
	// Base for the polynomial hash (number of characters in alphabet)
	Base = 256
)

func RabinKarpSearch(text, pattern string) int {
	n := len(text)
	m := len(pattern)
	if m == 0 {
		return 0
	}
	if m > n {
		return -1
	}

	var patternHash, textHash uint64
	var h uint64 = 1

	// Precompute h = pow(Base, m-1) % Prime
	for i := 0; i < m-1; i++ {
		h = (h * Base) % Prime
	}

	// Calculate initial hashes for pattern and first window of text
	for i := 0; i < m; i++ {
		patternHash = (Base*patternHash + uint64(pattern[i])) % Prime
		textHash = (Base*textHash + uint64(text[i])) % Prime
	}

	// Slide the window
	for i := 0; i <= n-m; i++ {
		// Check if hash values match
		if patternHash == textHash {
			// Confirm with character check (avoid spurious hits)
			if text[i:i+m] == pattern {
				return i
			}
		}

		// Calculate hash for next window
		if i < n-m {
			textHash = (Base*(textHash-uint64(text[i])*h) + uint64(text[i+m])) % Prime
			// We might get negative value, convert it to positive
			// Note: In Go uint64 handles wrapping, but logically in math:
			// if textHash < 0 { textHash += Prime }
		}
	}

	return -1
}

func main() {
	fmt.Println("Index:", RabinKarpSearch("ABABDABACDABABCABAB", "ABABCABAB"))
}
```

## Interview Questions

### Q: What is a "Spurious Hit"?
**A:** A spurious hit occurs when the hash value of the pattern matches the hash value of the current window, but the actual strings are different (a hash collision). The algorithm must verify the match character-by-character to ensure correctness.

### Q: Why is Rabin-Karp useful for multiple pattern search?
**A:** Rabin-Karp can be easily extended to search for **multiple patterns** of the same length simultaneously. You can store the hashes of all patterns in a set (or Bloom filter) and check the current window's hash against this set.

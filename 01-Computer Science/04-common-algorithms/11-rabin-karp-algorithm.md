---
---

# Rabin-Karp Algorithm

## Abstract
The **Rabin-Karp Algorithm** is a string searching algorithm that uses **hashing** to find a pattern in a text. Instead of checking every character, it compares the hash value of the pattern with the hash values of all substrings of the text. It is particularly effective for **multiple pattern search**.

## Development

### Core Concept: Rolling Hash
Calculating a hash for every substring naively would be $O(N \cdot M)$.
Rabin-Karp uses a **Rolling Hash** technique:
-   Hash of next window is calculated from the previous window in $O(1)$ time.
-   Formula: $H_{next} = (H_{prev} - \text{leading\_term}) \times \text{base} + \text{new\_char}$.

### Algorithm Steps
1.  Compute hash of Pattern.
2.  Compute hash of first window of Text.
3.  Slide window one char at a time:
    -   Update hash ($O(1)$).
    -   If Hash(Window) == Hash(Pattern):
        -   **Spurious Hit Check**: Verify characters one by one (to handle hash collisions).

### Complexity
-   **Time**:
    -   Average: $O(N + M)$.
    -   Worst: $O(N \cdot M)$ (if many hash collisions occur).
-   **Space**: $O(1)$.

## Code Examples (Go)

```go
package main

import "fmt"

const (
    base = 256
    mod  = 101 // Prime number
)

func RabinKarp(text, pattern string) []int {
    n, m := len(text), len(pattern)
    var matches []int
    
    if m > n { return nil }

    pHash, tHash := 0, 0
    h := 1

    // Precompute h = pow(base, m-1) % mod
    for i := 0; i < m-1; i++ {
        h = (h * base) % mod
    }

    // Calculate initial hashes
    for i := 0; i < m; i++ {
        pHash = (base*pHash + int(pattern[i])) % mod
        tHash = (base*tHash + int(text[i])) % mod
    }

    // Slide window
    for i := 0; i <= n-m; i++ {
        // Check hash match
        if pHash == tHash {
            // Check character match (handle collision)
            if text[i:i+m] == pattern {
                matches = append(matches, i)
            }
        }

        // Calculate next hash
        if i < n-m {
            tHash = (base*(tHash - int(text[i])*h) + int(text[i+m])) % mod
            if tHash < 0 {
                tHash += mod
            }
        }
    }
    return matches
}
```

## Go Application
-   **Plagiarism Detection**: Detecting matching sentence structures.
-   **Multiple Pattern Search**: Rabin-Karp can check for *any* of $k$ patterns simultaneously by storing pattern hashes in a Bloom Filter or Hash Map.

## Interview Preparation
1.  **Rolling Hash**: Explain the math ($O(1)$ update). Why use a prime modulus? (To minimize collisions).
2.  **Worst Case**: What if `base` and `mod` are chosen poorly? Many collisions $\to$ devolves to $O(N \cdot M)$ verification.

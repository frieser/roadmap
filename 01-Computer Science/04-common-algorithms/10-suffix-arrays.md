---
---

# Suffix Arrays

## Abstract
A **Suffix Array** is a sorted array of all suffixes of a string. It provides a space-efficient alternative to **Suffix Trees** for solving complex string problems like finding the longest repeated substring, locating a substring, or multiple pattern matching.

## Development

### Core Concept
Given a string $S$ of length $N$:
1.  Generate all $N$ suffixes.
    -   $S=$ "banana"
    -   Suffixes: "banana", "anana", "nana", "ana", "na", "a"
2.  Sort them lexicographically.
    -   Sorted: "a", "ana", "anana", "banana", "na", "nana"
3.  The Suffix Array stores the **starting indices** of these sorted suffixes.
    -   $SA = [5, 3, 1, 0, 4, 2]$

### Construction Complexity
-   **Naive**: $O(N^2 \log N)$ using standard sort comparison (string compare takes $O(N)$).
-   **Optimized**: $O(N \log N)$ or even $O(N)$ (e.g., SA-IS algorithm).

### Applications
-   **Pattern Searching**: Use Binary Search on the suffix array. Time: $O(M \log N)$.
-   **LCP Array (Longest Common Prefix)**: Auxiliary array used with SA to solve "Longest Repeated Substring" in $O(N)$.

## Code Examples (Go)

### Using `index/suffixarray`
Go provides a high-performance implementation in the standard library.

```go
package main

import (
    "fmt"
    "index/suffixarray"
    "sort"
)

func main() {
    text := "banana"
    
    // 1. Efficient Construction (O(N) or O(N log N))
    sa := suffixarray.New([]byte(text))

    // 2. Lookup (Find all occurrences)
    offsets := sa.Lookup([]byte("ana"), -1)
    fmt.Println("Occurrences of 'ana':", offsets) // [1 3]
    
    // 3. Manual Construction (Naive O(N^2 log N) for understanding)
    naiveSA := make([]int, len(text))
    for i := range naiveSA {
        naiveSA[i] = i
    }
    sort.Slice(naiveSA, func(i, j int) bool {
        return text[naiveSA[i]:] < text[naiveSA[j]:]
    })
    fmt.Println("Suffix Array:", naiveSA)
}
```

## Go Application
-   **Bioinformatics**: DNA sequence analysis (genome assembly).
-   **Full-Text Search**: Building indices for large static text corpora where fast substring queries are needed.

## Interview Preparation
1.  **Suffix Tree vs Suffix Array**:
    -   Tree: $O(N)$ construction, easy to conceptualize, but high memory overhead (pointers).
    -   Array: $O(N)$ or $O(N \log N)$ construction, minimal memory ($O(N)$ integers), but requires LCP array for advanced queries.
2.  **LCP Array**: Stores the length of the longest common prefix between consecutive suffixes in the sorted array. Essential for finding the longest repeated substring.

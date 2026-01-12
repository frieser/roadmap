---
---

# Search Pattern in Text

## Abstract
**Pattern Searching** is the problem of finding a given substring (pattern) within a larger string (text). While a naive approach works for simple cases, specialized algorithms like **Knuth-Morris-Pratt (KMP)**, **Rabin-Karp**, and **Boyer-Moore** offer significant performance improvements by skipping unnecessary comparisons.

## Development

### Naive Approach
Iterate through the text. For each position, check if the pattern matches starting from that position.
-   **Time Complexity**: $O((N-M+1) \cdot M)$. Worst case happens with inputs like Text="AAAAA...B", Pattern="AAAAB".
-   **Space Complexity**: $O(1)$.

### Knuth-Morris-Pratt (KMP)
KMP improves upon the naive approach by observing that when a mismatch occurs, the pattern itself embodies sufficient information to determine where the next match could begin, thus bypassing re-examination of previously matched characters.
-   **LPS Array**: "Longest Prefix Suffix". Stores the length of the longest proper prefix which is also a suffix.
-   **Complexity**: $O(N + M)$.

### Go Standard Library (`strings.Index`)
Go's implementation is highly optimized.
-   For small strings, it uses a brute-force or Rabin-Karp-like specialized approach.
-   For larger strings (and CPU support), it uses SIMD instructions (AVX2/SSE) via assembly for extreme speed.
-   It is often faster than a manual KMP implementation in Go due to assembly optimizations.

## Code Examples (Go)

### 1. Naive Implementation
```go
func NaiveSearch(text, pattern string) []int {
    n, m := len(text), len(pattern)
    var matches []int

    for i := 0; i <= n-m; i++ {
        match := true
        for j := 0; j < m; j++ {
            if text[i+j] != pattern[j] {
                match = false
                break
            }
        }
        if match {
            matches = append(matches, i)
        }
    }
    return matches
}
```

### 2. KMP Implementation
```go
func KMPSearch(text, pattern string) []int {
    n, m := len(text), len(pattern)
    if m == 0 { return nil }
    
    lps := computeLPS(pattern)
    var matches []int
    i, j := 0, 0 // i for text, j for pattern

    for i < n {
        if pattern[j] == text[i] {
            i++
            j++
        }
        if j == m {
            matches = append(matches, i-j)
            j = lps[j-1]
        } else if i < n && pattern[j] != text[i] {
            if j != 0 {
                j = lps[j-1]
            } else {
                i++
            }
        }
    }
    return matches
}

func computeLPS(pattern string) []int {
    m := len(pattern)
    lps := make([]int, m)
    len := 0
    i := 1
    
    for i < m {
        if pattern[i] == pattern[len] {
            len++
            lps[i] = len
            i++
        } else {
            if len != 0 {
                len = lps[len-1]
            } else {
                lps[i] = 0
                i++
            }
        }
    }
    return lps
}
```

## Go Application
-   **`strings.Contains` / `strings.Index`**: The daily driver. Always prefer these over manual implementations unless you have a specific reason (e.g., streaming data, multiple patterns).
-   **Log Analysis**: Searching for error codes in massive log files.

## Interview Preparation
1.  **Worst Case of Naive Search**: $O(N \cdot M)$. Occurs when pattern matches text partially many times (e.g., T="AAAA...", P="AAAB").
2.  **Why KMP is O(N)?**: The text pointer `i` never moves back. The pattern pointer `j` moves back intelligently using the LPS array.

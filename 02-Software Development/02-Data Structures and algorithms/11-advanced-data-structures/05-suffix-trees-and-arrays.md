---
---

# Suffix Trees and Arrays

## Summary
**Suffix Trees** and **Suffix Arrays** are advanced data structures used to solve string problems involving patterns and substrings efficiently. They represent **all suffixes** of a string in a sorted or tree-like manner.

## Detailed Explanation

### Suffix Tree
A compressed Trie of all suffixes of a text.
*   **Edges**: Labeled with substrings.
*   **Nodes**: Represent matching substrings.
*   **Power**: Can solve "Is pattern P in text T?" in $O(|P|)$ time (independent of $|T|$ after construction).
*   **Drawback**: Huge memory consumption (~20x the text size) and complex construction ($O(N)$ via Ukkonen's Algorithm).

### Suffix Array
A sorted array of all integers representing the starting indices of all suffixes of a string.
*   Text: "BANANA"
*   Suffixes: "BANANA", "ANANA", "NANA", "ANA", "NA", "A"
*   Sorted: "A", "ANA", "ANANA", "BANANA", "NA", "NANA"
*   Indices: `[5, 3, 1, 0, 4, 2]`
*   **Advantage**: Much more space-efficient than Suffix Trees ($O(N)$ integers).

### LCP Array (Longest Common Prefix)
Often stored with Suffix Arrays. `LCP[i]` stores the length of the common prefix between `Suffix[i]` and `Suffix[i-1]`.
*   Combined with Suffix Array, it replicates the power of a Suffix Tree.

## Complexity (Suffix Array)
| Operation | Time | Notes |
| :--- | :--- | :--- |
| **Build** | $O(n \log n)$ or $O(n)$ | $O(n)$ is complex (SA-IS alg), $O(n \log^2 n)$ is common. |
| **Search** | $O(P \log n)$ | Using Binary Search on the array. |

## Code Examples (Go)
*Simple $O(n^2 \log n)$ construction for educational purpose.*

```go
package main

import (
    "fmt"
    "sort"
    "strings"
)

func BuildSuffixArray(s string) []int {
    n := len(s)
    suffixes := make([]int, n)
    for i := 0; i < n; i++ {
        suffixes[i] = i
    }
    
    sort.Slice(suffixes, func(i, j int) bool {
        // Compare substrings starting at suffixes[i] and suffixes[j]
        return strings.Compare(s[suffixes[i]:], s[suffixes[j]:]) < 0
    })
    
    return suffixes
}

func main() {
    fmt.Println(BuildSuffixArray("banana"))
}
```

## Go Application
*   **Bioinformatics**: DNA sequence alignment.
*   **Compression**: Burrows-Wheeler Transform (BWT) used in bzip2.
*   **Full Text Search**: Finding longest repeated substrings.

## Interview Questions

**Q: Suffix Tree vs Suffix Array?**
**A:**
*   **Suffix Tree**: Faster for complex traversals ($O(N)$ construction), but uses excessive memory (pointers, nodes).
*   **Suffix Array**: Slower construction in practice (usually $O(N \log N)$), but extremely memory efficient ($O(N)$ integers). Preferred in competitive programming and memory-constrained systems.

**Q: How do you find the Longest Repeated Substring?**
**A:** Construct the Suffix Array and the LCP Array. The maximum value in the LCP Array corresponds to the longest repeated substring.

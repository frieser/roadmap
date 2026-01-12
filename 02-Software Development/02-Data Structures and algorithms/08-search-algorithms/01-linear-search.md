---
---

# Linear Search

## Summary
**Linear Search** (or Sequential Search) is the simplest searching algorithm. It works by iterating through a collection element by element until the target value is found or the end of the list is reached. It requires no preprocessing (like sorting) but is inefficient for large datasets.

## Detailed Explanation

### Mechanism
1.  Start at the first element (index 0).
2.  Compare the current element with the target.
3.  If they match, return the current index.
4.  If not, move to the next element.
5.  If the end of the list is reached without a match, return a "not found" indicator (e.g., `-1`).

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Best)** | $O(1)$ | Target is the first element. |
| **Time (Worst)** | $O(n)$ | Target is the last element or not present. |
| **Time (Avg)** | $O(n)$ | Target is somewhere in the middle. |
| **Space** | $O(1)$ | No extra memory required. |

## Code Examples (Go)

### 1. Basic Implementation
```go
package main

import "fmt"

// LinearSearch returns the index of target or -1 if not found
func LinearSearch(arr []int, target int) int {
    for i, val := range arr {
        if val == target {
            return i
        }
    }
    return -1
}

func main() {
    items := []int{10, 50, 30, 70, 80, 20}
    idx := LinearSearch(items, 30)
    fmt.Printf("Found at index: %d\n", idx)
}
```

### 2. Standard Library (Go 1.21+)
The `slices` package provides optimized linear search functions.

```go
import "slices"

func main() {
    items := []string{"apple", "banana", "cherry"}
    
    // Check existence (returns bool)
    exists := slices.Contains(items, "banana")
    
    // Find index (returns index or -1)
    idx := slices.Index(items, "cherry")
}
```

## Go Application
*   **Small Datasets**: For small lists (e.g., < 50 items), Linear Search is often faster than Binary Search due to CPU cache locality and the lack of sorting overhead.
*   **Unsorted Data**: It is the *only* option when data cannot be sorted or when sorting ($O(n \log n)$) would take longer than the search itself.

## Interview Questions

**Q: When would you use Linear Search over Binary Search?**
**A:**
1.  When the data is **unsorted**.
2.  When the dataset is **small** enough that the overhead of sorting is unjustified.
3.  When the data structure does not support random access (e.g., a Singly Linked List).

**Q: Can Linear Search be recursive?**
**A:** Yes, but it adds $O(n)$ space complexity for the call stack without improving time complexity, making it a bad practice compared to the iterative approach.

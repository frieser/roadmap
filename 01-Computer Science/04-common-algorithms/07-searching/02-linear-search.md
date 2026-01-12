---
---

# Linear Search

## Abstract
**Linear Search** (or Sequential Search) is the simplest searching algorithm. It checks every element in the list sequentially until the desired element is found or the list ends. It works on both sorted and unsorted lists.

## Development

### Core Concept
1.  Start from the leftmost element `arr[0]`.
2.  Compare `arr[i]` with target.
3.  If match, return index.
4.  If no match, move to `arr[i+1]`.
5.  If end reached, return -1.

### Complexity
-   **Time**: $O(n)$.
-   **Space**: $O(1)$.

## Code Examples (Go)

```go
package main

import "fmt"

func LinearSearch(arr []int, target int) int {
    for i, v := range arr {
        if v == target {
            return i
        }
    }
    return -1
}

func main() {
    items := []int{9, 4, 2, 7, 1}
    fmt.Println(LinearSearch(items, 7)) // 3
}
```

### Sentinel Optimization
To avoid checking `i < len(arr)` in every iteration, place the target at the end of the array (if mutable/expandable) as a "sentinel". The loop is guaranteed to terminate.

```go
func LinearSearchSentinel(arr []int, target int) int {
    n := len(arr)
    last := arr[n-1]
    
    // If target is last, return immediately
    if last == target {
        return n - 1
    }
    
    // Replace last with target (Sentinel)
    arr[n-1] = target
    i := 0
    for arr[i] != target {
        i++
    }
    
    // Restore
    arr[n-1] = last
    
    if i < n-1 {
        return i
    }
    return -1
}
```

## Go Application
-   **Small Datasets**: For very small arrays ($n < 50$), linear search can be faster than binary search due to branch prediction and CPU caching (no random jumps).
-   **Unsorted Data**: The only option if data cannot be sorted.

## Interview Preparation
1.  **When is Linear better than Binary?**: When the array is unsorted, or extremely small, or linked lists (where random access is not possible).
2.  **Worst Case**: Element is at the end or not present ($n$ comparisons).

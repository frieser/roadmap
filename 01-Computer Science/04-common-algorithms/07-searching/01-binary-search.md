---
---

# Binary Search

## Abstract
**Binary Search** is an efficient algorithm for finding an item from a **sorted** list of items. It works by repeatedly dividing in half the portion of the list that could contain the item, until you've narrowed down the possible locations to just one.

## Development

### Core Concept
1.  **Compare** the target value to the middle element of the array.
2.  If they are equal, the search is successful.
3.  If the target is smaller, search the **left** half.
4.  If the target is larger, search the **right** half.
5.  Repeat until found or subarray size is 0.

### Complexity
-   **Time**: $O(\log n)$.
-   **Space**: $O(1)$ (Iterative).

## Code Examples (Go)

### Standard Library (`sort.Search`)
Go provides a generic binary search that returns the first index $i$ in $[0, n)$ where $f(i)$ is true.

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    data := []int{1, 2, 3, 4, 5, 6, 7, 8}
    target := 5

    // sort.Search returns the smallest index i such that f(i) is true
    i := sort.Search(len(data), func(i int) bool {
        return data[i] >= target
    })

    if i < len(data) && data[i] == target {
        fmt.Printf("Found %d at index %d\n", target, i)
    } else {
        fmt.Printf("%d not found\n", target)
    }
}
```

### Manual Implementation
```go
func BinarySearch(arr []int, target int) int {
    low, high := 0, len(arr)-1
    
    for low <= high {
        // Prevent overflow: low + (high-low)/2
        mid := int(uint(low+high) >> 1) 
        
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return -1
}
```

## Go Application
-   **Database Indexes**: Finding records in B-Trees.
-   **Debugging**: Git Bisect uses binary search to find the commit that introduced a bug.

## Interview Preparation
1.  **Pre-condition**: Array MUST be sorted.
2.  **Integer Overflow**: Calculating `(low + high) / 2` can overflow if indices are large. Use `low + (high - low) / 2`.
3.  **Lower/Upper Bound**: `sort.Search` effectively finds the Lower Bound (first occurrence).

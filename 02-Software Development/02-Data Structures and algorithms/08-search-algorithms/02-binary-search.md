---
---

# Binary Search

## Summary
**Binary Search** is a highly efficient search algorithm that works on **sorted** arrays. It follows the **Divide and Conquer** paradigm: repeatedly dividing the search interval in half. If the target value is less than the middle element, the search continues in the lower half, otherwise in the upper half.

## Detailed Explanation

### Mechanism
1.  Set `low` to 0 and `high` to $N-1$.
2.  While `low <= high`:
    *   Calculate `mid = low + (high - low) / 2`.
    *   If `arr[mid] == target`, return `mid`.
    *   If `arr[mid] < target`, discard the left half (`low = mid + 1`).
    *   If `arr[mid] > target`, discard the right half (`high = mid - 1`).
3.  If loop finishes, return "not found".

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Best)** | $O(1)$ | Target is exactly at the middle. |
| **Time (Worst)** | $O(\log n)$ | Reducing search space by half each step. |
| **Time (Avg)** | $O(\log n)$ | |
| **Space** | $O(1)$ | Iterative implementation. |

## Code Examples (Go)

### 1. Iterative Implementation (Recommended)
```go
package main

import "fmt"

func BinarySearch(arr []int, target int) int {
    low := 0
    high := len(arr) - 1

    for low <= high {
        // Safe mid calculation to prevent integer overflow
        mid := low + (high-low)/2

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

### 2. Standard Library (Go 1.21+)
Go provides a generic binary search in the `slices` package.

```go
import (
    "fmt"
    "slices"
)

func main() {
    data := []int{1, 3, 5, 7, 9, 11} // MUST be sorted
    
    // Returns index where it IS or where it SHOULD be inserted
    idx, found := slices.BinarySearch(data, 7)
    
    if found {
        fmt.Printf("Found 7 at index %d\n", idx)
    } else {
        fmt.Printf("7 not found, insert at %d\n", idx)
    }
}
```

## Go Application
*   **Database Indexing**: Databases use B-Trees (a generalization of binary search) to quickly locate records.
*   **`sort.Search`**: Prior to Go 1.21, `sort.Search(n, f)` was used. It returns the smallest index `i` such that `f(i)` is true. It is powerful but slightly unintuitive compared to `slices.BinarySearch`.

## Interview Questions

**Q: Why do we calculate mid as `low + (high - low) / 2` instead of `(low + high) / 2`?**
**A:** To prevent **Integer Overflow**. If `low` and `high` are both large positive integers (near the max limit of `int`), adding them together might wrap around to a negative number, causing an index out of bounds error.

**Q: Does Binary Search work on Linked Lists?**
**A:** Not efficiently. Binary Search requires **random access** ($O(1)$) to jump to the middle. Linked Lists only support sequential access. Attempting Binary Search on a Linked List results in $O(n)$ complexity because getting to the middle takes $n/2$ steps.

**Q: What is the requirement for Binary Search?**
**A:** The data **must be sorted** monotonically (increasing or decreasing).

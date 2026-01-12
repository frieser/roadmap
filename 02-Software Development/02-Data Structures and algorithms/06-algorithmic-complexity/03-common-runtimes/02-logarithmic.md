---
---

# O(log n) Logarithmic Time

## Summary
**$O(\log n)$ Logarithmic Time** represents highly efficient algorithms that slow down very gradually as input grows. It typically appears in **"Divide and Conquer"** algorithms where the problem space is cut in half (or by a factor) at every step.

## Characteristics
*   **Scalability**: Excellent. Doubling the input size only adds **1** extra step.
    *   $N=1,000 \to 10$ steps.
    *   $N=1,000,000 \to 20$ steps.
*   **Mechanism**: Binary splitting, Trees, Heaps.
*   **Graph**: A curve that flattens out quickly.

## Common Operations
1.  **Binary Search**: Finding an item in a sorted list.
2.  **Balanced Search Trees**: Operations in BST, AVL, Red-Black Trees.
3.  **Heap Operations**: Insertion/Deletion in a Priority Queue.

## Go Code Examples

### Binary Search
The classic example. We search for a `target` in a sorted slice.

```go
package main

import "fmt"

// BinarySearch is O(log n)
func BinarySearch(arr []int, target int) int {
    low := 0
    high := len(arr) - 1

    for low <= high {
        // Calculate mid point
        mid := low + (high-low)/2 // Safe from overflow

        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            // Discard left half
            low = mid + 1
        } else {
            // Discard right half
            high = mid - 1
        }
    }
    return -1
}

func main() {
    // Array MUST be sorted
    data := []int{1, 3, 5, 7, 9, 11, 13, 15} 
    idx := BinarySearch(data, 13)
    fmt.Println("Index:", idx) // 6
}
```

### Understanding the Math
Why Log?
*   Iteration 1: $N$ items.
*   Iteration 2: $N/2$ items.
*   Iteration 3: $N/4$ items.
*   ...
*   Iteration $k$: $N / 2^k = 1$.

Solving for $k$: $2^k = N \implies k = \log_2 N$.

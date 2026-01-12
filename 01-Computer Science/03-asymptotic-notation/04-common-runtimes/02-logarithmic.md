---
---

# Logarithmic Time - O(log n)

## Abstract
**Logarithmic Time**, denoted as **O(log n)**, represents a runtime that grows very slowly as the input size ($n$) increases. It typically occurs in algorithms that **divide the problem in half** at each step. It is extremely efficient; even for massive inputs (e.g., 4 billion items), an O(log n) algorithm might only take ~32 steps.

## Development

### Core Concept
In mathematics, $\log_2 n = x$ means $2^x = n$. 
- If $n = 16$, $\log n = 4$ steps ($16 \to 8 \to 4 \to 2 \to 1$).
- If $n = 1,000,000$, $\log n \approx 20$ steps.
- If $n = 1,000,000,000$, $\log n \approx 30$ steps.

### Common Sources
- **Binary Search**: Searching in a sorted list.
- **Balanced Search Trees**: Operations in AVL, Red-Black trees.
- **Heaps**: Push/Pop in a priority queue.
- **Divide and Conquer** algorithms (the "divide" part).

## Code Examples (Go)

### 1. Binary Search
The classic example. Go's `sort` package provides this generic logic.

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    // Sorted array is required
    nums := []int{2, 4, 6, 8, 10, 12, 14, 16}
    target := 10

    // sort.Search uses binary search logic
    // Returns the smallest index i where f(i) is true
    idx := sort.Search(len(nums), func(i int) bool {
        return nums[i] >= target
    })

    if idx < len(nums) && nums[idx] == target {
        fmt.Printf("Found %d at index %d\n", target, idx)
    } else {
        fmt.Println("Not found")
    }
}
```

### 2. Manual Binary Search
```go
func binarySearch(nums []int, target int) int {
    low, high := 0, len(nums)-1
    
    for low <= high {
        // Safe middle calculation to avoid overflow
        mid := low + (high-low)/2 
        
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return -1
}
```

## Go Application & Ecosystem
- **`sort.Search`**: Used extensively in Go for finding items in sorted slices.
- **`container/heap`**: Pushing and Popping items is O(log n).
- **Database Indexes**: B-Tree lookups (used in SQL databases) are logarithmic.

## Interview Preparation
1.  **Why is Binary Search O(log n)?**
    -   *Answer*: Because we discard half the search space in every iteration. The input size $n$ is reduced to $n/2, n/4, \dots, 1$.
2.  **Does Binary Search work on unsorted arrays?**
    -   *Answer*: No. The data must be sorted first (which takes O(n log n)).
3.  **What if I double the input size?**
    -   *Answer*: The runtime only increases by **1 step** (constant amount).

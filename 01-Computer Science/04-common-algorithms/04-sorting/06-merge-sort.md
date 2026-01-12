---
---

# Merge Sort

## Abstract
**Merge Sort** is a Divide and Conquer algorithm. It divides the input array into two halves, calls itself for the two halves, and then **merges** the two sorted halves. It is one of the most efficient sorting algorithms for large datasets and linked lists.

## Development

### Core Concept
1.  **Divide**: Find middle index `m`.
2.  **Conquer**: Recursively sort `arr[0...m]` and `arr[m+1...n]`.
3.  **Combine**: Merge the two sorted subarrays into a single sorted result.

### Complexity
-   **Time**: $O(n \log n)$ **guaranteed** (Worst, Best, Average).
-   **Space**: $O(n)$ (Requires auxiliary array for merging).
-   **Stability**: **Stable**.

## Code Examples (Go)

```go
package main

import "fmt"

func MergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }

    mid := len(arr) / 2
    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])

    return merge(left, right)
}

func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0

    for i < len(left) && j < len(right) {
        if left[i] <= right[j] { // <= ensures Stability
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }

    // Append remaining elements
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    return result
}

func main() {
    arr := []int{12, 11, 13, 5, 6, 7}
    sorted := MergeSort(arr)
    fmt.Println(sorted)
}
```

## Go Application
- **Stable Sort**: Go's `sort.Stable` usually implements a form of Merge Sort (or SymMerge) because Quick Sort is unstable.
- **Linked Lists**: Merge sort is the preferred algorithm for sorting linked lists ($O(1)$ space possible).

## Interview Preparation
1.  **Space Complexity**: Why is it $O(n)$?
    -   *Answer*: We need a temporary array to hold the merged elements. In-place merge sort exists but is complex and slow.
2.  **When to use Merge Sort?**: When stability is required or sorting Linked Lists.

---
---

# Quick Sort

## Abstract
**Quick Sort** is a highly efficient sorting algorithm and is based on the **Divide and Conquer** paradigm. It works by selecting a 'pivot' element from the array and partitioning the other elements into two sub-arrays, according to whether they are less than or greater than the pivot.

## Development

### Core Concept
1.  **Pivot**: Pick an element (First, Last, Random, or Median).
2.  **Partition**: Reorder array so all elements < pivot come before pivot, and all > pivot come after.
3.  **Recurse**: Apply recursively to sub-arrays.

### Partition Schemes
-   **Lomuto**: Simpler code, more swaps.
-   **Hoare**: Fewer swaps, slightly faster, trickier indices.

### Complexity
-   **Time**:
    -   Average: $O(n \log n)$.
    -   Worst: $O(n^2)$ (e.g., sorted array with first element as pivot).
-   **Space**: $O(\log n)$ (Recursion stack).
-   **Stability**: **Unstable**.

## Code Examples (Go)

### Standard Lomuto Implementation
```go
package main

import "fmt"

func QuickSort(arr []int, low, high int) {
    if low < high {
        p := partition(arr, low, high)
        QuickSort(arr, low, p-1)
        QuickSort(arr, p+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high] // Last element as pivot
    i := low - 1       // Index of smaller element

    for j := low; j < high; j++ {
        if arr[j] < pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}

func main() {
    arr := []int{10, 7, 8, 9, 1, 5}
    QuickSort(arr, 0, len(arr)-1)
    fmt.Println(arr)
}
```

## Go Application
- **`sort` Package**: Go 1.19+ uses **pdqsort** (Pattern-Defeating Quicksort), which is a sophisticated Quick Sort variant that detects patterns (sorted, reverse) to be $O(n)$ and falls back to Heap Sort if recursion goes too deep.

## Interview Preparation
1.  **Worst Case Mitigation**: How to avoid $O(n^2)$?
    -   *Answer*: Pick random pivot or "Median of 3" pivot.
2.  **Why Quick Sort > Merge Sort?**: Better locality of reference (cache friendly) and requires less memory ($O(\log n)$ vs $O(n)$).

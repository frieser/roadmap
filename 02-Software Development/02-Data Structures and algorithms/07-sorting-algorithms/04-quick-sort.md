---
---

# Quick Sort

## Summary
**Quick Sort** is an efficient, **Divide and Conquer** algorithm and is the industry standard for general-purpose sorting. It works by selecting a "pivot" element and partitioning the array so that all elements smaller than the pivot come before it, and all greater come after. It is typically **unstable** but runs fast in practice due to good cache locality.

## Detailed Explanation

### Mechanism
1.  **Pivot Selection**: Choose an element (first, last, random, or median-of-three).
2.  **Partitioning**: Reorder the array so that `left < pivot < right`.
3.  **Recursion**: Apply the same logic to the sub-arrays `left` and `right`.
4.  **Base Case**: Arrays of size 0 or 1 are sorted.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Avg)** | $O(n \log n)$ | |
| **Time (Worst)** | $O(n^2)$ | Occurs if pivot is always the smallest/largest (e.g., sorted array). |
| **Space** | $O(\log n)$ | Stack space for recursion. |
| **Stable?** | No | Swapping can disrupt relative order. |

## Code Examples (Go)

```go
package main

import "fmt"

func QuickSort(arr []int, low, high int) {
    if low < high {
        // partitionIndex is partitioning index, arr[p] is now at right place
        p := partition(arr, low, high)
        
        QuickSort(arr, low, p-1)
        QuickSort(arr, p+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high] // Choosing last element as pivot
    i := low - 1       // Index of smaller element
    
    for j := low; j < high; j++ {
        if arr[j] < pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    // Swap pivot to correct position
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}

func main() {
    data := []int{10, 7, 8, 9, 1, 5}
    QuickSort(data, 0, len(data)-1)
    fmt.Println(data)
}
```

## Go Application: `pdqsort`
Since Go 1.19, the standard library function `slices.Sort` (and `sort.Sort` internally) uses **pdqsort** (Pattern-Defeating Quicksort).
*   **Hybrid**: It combines Quick Sort (fast), Heap Sort (fallback to avoid $O(n^2)$ worst case), and Insertion Sort (for small partitions).
*   **Adaptive**: It detects if the array is already sorted or reverse sorted to run in $O(n)$.
*   **Unstable**: If you need stability, use `slices.SortStable`.

## Interview Questions

**Q: What is the worst-case time complexity of Quick Sort, and how do we avoid it?**
**A:** The worst case is $O(n^2)$, which happens if the pivot is consistently the smallest or largest element (e.g., sorting an already sorted array with the first element as pivot). We avoid this by:
1.  Choosing a random pivot.
2.  Using "Median-of-Three" pivot selection.
3.  Switching to Heap Sort if recursion depth gets too deep (Introsort strategy).

**Q: Why is Quick Sort faster than Merge Sort in practice?**
**A:**
1.  **Cache Locality**: Quick Sort accesses memory linearly during partitioning, maximizing cache hits.
2.  **No Extra Space**: It runs in-place (mostly), avoiding the allocation overhead of Merge Sort.

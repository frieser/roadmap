---
---

# Insertion Sort

## Abstract
**Insertion Sort** works the way you sort playing cards in your hands. It iterates through the input elements and at each iteration, removes one element from the input data, finds the location it belongs to within the sorted list, and inserts it there.

## Development

### Core Concept
1.  Assume `arr[0]` is sorted.
2.  Take `arr[1]`. If smaller than `arr[0]`, shift `arr[0]` right and place `arr[1]`.
3.  Take `arr[i]`. Scan backwards from `i-1` to 0. Shift elements greater than `arr[i]` to the right. Place `arr[i]` in the gap.

### Complexity
- **Time**: 
    -   Worst/Average: $O(n^2)$.
    -   Best: $O(n)$ (Already sorted).
- **Space**: $O(1)$.
- **Stability**: **Stable**.

## Code Examples (Go)

```go
package main

import "fmt"

func InsertionSort(arr []int) {
    for i := 1; i < len(arr); i++ {
        key := arr[i]
        j := i - 1

        // Move elements of arr[0..i-1], that are greater than key,
        // to one position ahead of their current position
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j = j - 1
        }
        arr[j+1] = key
    }
}

func main() {
    arr := []int{12, 11, 13, 5, 6}
    InsertionSort(arr)
    fmt.Println(arr)
}
```

## Go Application
- **Small Arrays**: Insertion sort is very fast for small arrays (e.g., $n < 20$) due to low overhead.
- **Hybrid Algorithms**: Go's `sort` package (and many standard libraries) uses **pdqsort** (Pattern-Defeating Quicksort), which switches to Insertion Sort for small partitions. It is a critical component of high-performance sorting.

## Interview Preparation
1.  **Online Algorithm**: Insertion sort can sort a list as it receives it (streaming), unlike Quick/Merge sort which usually need all data.
2.  **Almost Sorted**: If the array is "almost sorted" (few inversions), Insertion Sort is $O(n)$, beating Quick Sort.

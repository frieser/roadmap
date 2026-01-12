---
---

# Selection Sort

## Abstract
**Selection Sort** sorts an array by repeatedly finding the **minimum element** (considering ascending order) from the unsorted part and putting it at the beginning. It maintains two subarrays: the sorted part at the left end and the unsorted part at the right end.

## Development

### Core Concept
1.  Find the minimum element in `arr[0...n-1]`.
2.  Swap it with `arr[0]`.
3.  Find the minimum in `arr[1...n-1]`.
4.  Swap it with `arr[1]`.
5.  Repeat until unsorted part is empty.

### Complexity
- **Time**: $O(n^2)$ for ALL cases (Best, Worst, Average). It always scans the remaining array.
- **Space**: $O(1)$ (In-place).
- **Stability**: **Unstable** (Swapping long-distance can reorder equal elements).

## Code Examples (Go)

```go
package main

import "fmt"

func SelectionSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        minIdx := i
        for j := i + 1; j < n; j++ {
            if arr[j] < arr[minIdx] {
                minIdx = j
            }
        }
        // Swap the found minimum element with the first element
        arr[i], arr[minIdx] = arr[minIdx], arr[i]
    }
}

func main() {
    arr := []int{64, 25, 12, 22, 11}
    SelectionSort(arr)
    fmt.Println(arr)
}
```

## Go Application
- **Minimize Swaps**: Selection Sort makes the minimum number of swaps ($O(n)$) compared to Bubble/Insertion sort. It might be useful if **writing to memory** is extremely expensive (e.g., flash memory), though this is a niche case.

## Interview Preparation
1.  **Selection vs Bubble**: Selection is generally faster than Bubble because it makes fewer swaps, but both are $O(n^2)$ comparisons.
2.  **Stability**: Example of instability: `[5a, 5b, 1]`. Smallest is 1. Swap 1 with 5a. Result `[1, 5b, 5a]`. Order of 5s changed.

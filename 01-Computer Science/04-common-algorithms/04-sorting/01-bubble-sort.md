---
---

# Bubble Sort

## Abstract
**Bubble Sort** is the simplest sorting algorithm that works by repeatedly swapping the adjacent elements if they are in the wrong order. It "bubbles" the largest element to the end of the array in each iteration. It is primarily educational and almost never used in production due to its poor performance.

## Development

### Core Concept
1.  Iterate through the array from 0 to $n-1$.
2.  For each element, compare with next neighbor.
3.  Swap if `arr[i] > arr[i+1]`.
4.  Repeat $n$ times.
5.  Optimization: If no swaps occurred in a pass, the array is sorted (early exit).

### Complexity
- **Time**: 
    -   Worst/Average: $O(n^2)$
    -   Best: $O(n)$ (if already sorted and optimized flag used).
- **Space**: $O(1)$ (In-place).
- **Stability**: **Stable** (Does not swap equal elements).

## Code Examples (Go)

```go
package main

import "fmt"

func BubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        swapped := false
        // Last i elements are already in place
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = true
            }
        }
        // Optimization: Stop if no swaps happened
        if !swapped {
            break
        }
    }
}

func main() {
    arr := []int{64, 34, 25, 12, 22, 11, 90}
    BubbleSort(arr)
    fmt.Println(arr)
}
```

## Go Application
- **None**: Never use this in real Go code. Use `sort.Slice` or `slices.Sort`.

## Interview Preparation
1.  **Why is it called Bubble Sort?**: Large elements "bubble" to the top (end) of the list like air bubbles in water.
2.  **When is it useful?**: Only for extremely small inputs ($n < 10$) or for checking if an array is already sorted (O(n) best case), though `IsSorted` functions are better.

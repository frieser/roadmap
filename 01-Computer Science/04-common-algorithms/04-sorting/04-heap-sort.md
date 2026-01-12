---
---

# Heap Sort

## Abstract
**Heap Sort** is a comparison-based sorting algorithm that uses a **Binary Heap** data structure. It divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element and moving that to the sorted region.

## Development

### Core Concept
1.  **Build Max Heap**: Transform the array into a Max Heap ($O(n)$).
2.  **Extract Max**: The largest item is at the root (`arr[0]`). Swap it with the last item (`arr[end]`).
3.  **Heapify**: Reduce heap size by 1 and restore heap property for the root.
4.  Repeat until heap size is 1.

### Complexity
- **Time**: $O(n \log n)$ for all cases.
- **Space**: $O(1)$ (In-place).
- **Stability**: **Unstable**.

## Code Examples (Go)

Using `container/heap` is possible, but Heap Sort is usually implemented raw on arrays for performance/in-place constraint.

```go
package main

import "fmt"

func HeapSort(arr []int) {
    n := len(arr)

    // Build max heap
    for i := n/2 - 1; i >= 0; i-- {
        heapify(arr, n, i)
    }

    // Extract elements one by one
    for i := n - 1; i > 0; i-- {
        // Move current root to end
        arr[0], arr[i] = arr[i], arr[0]

        // Call max heapify on the reduced heap
        heapify(arr, i, 0)
    }
}

func heapify(arr []int, n int, i int) {
    largest := i
    l := 2*i + 1
    r := 2*i + 2

    if l < n && arr[l] > arr[largest] {
        largest = l
    }

    if r < n && arr[r] > arr[largest] {
        largest = r
    }

    if largest != i {
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
    }
}

func main() {
    arr := []int{12, 11, 13, 5, 6, 7}
    HeapSort(arr)
    fmt.Println(arr)
}
```

## Go Application
- **Memory Constrained**: Good when $O(1)$ space is strict and $O(n \log n)$ time is required.
- **Worst-Case Guarantee**: Used as a fallback in Quick Sort implementations (Introsort) to prevent $O(n^2)$ worst case.

## Interview Preparation
1.  **Comparison to Quick Sort**: Heap Sort is slower in practice (constants factors) due to poor cache locality (jumping around array indices `2*i`).
2.  **Construction**: Building a heap is $O(n)$, not $O(n \log n)$. Sorting is $O(n \log n)$.

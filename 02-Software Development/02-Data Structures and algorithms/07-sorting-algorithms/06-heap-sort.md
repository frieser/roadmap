---
---

# Heap Sort

## Summary
**Heap Sort** is a comparison-based sorting algorithm that uses a **Binary Heap** data structure. It divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element from it and moving that to the sorted region. It is efficient and requires no extra memory.

## Detailed Explanation

### Mechanism
1.  **Build Max Heap**: Rearrange the array so it satisfies the Max-Heap property (Parent $\ge$ Children). The largest element is now at `arr[0]`.
2.  **Extract**: Swap `arr[0]` (max) with the last element `arr[end]`.
3.  **Reduce Heap**: Decrease the considered heap size by 1.
4.  **Heapify**: Restore the Max-Heap property for the remaining elements at the root.
5.  Repeat until heap size is 1.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (All)** | $O(n \log n)$ | Building heap is $O(n)$, extracting is $n \times \log n$. |
| **Space** | $O(1)$ | In-place. |
| **Stable?** | No | The structure of the heap shuffling destroys stability. |

## Code Examples (Go)

### Idiomatic Usage (`container/heap`)
Go provides a heap interface. Using it for sorting:

```go
package main

import (
    "container/heap"
    "fmt"
)

// IntHeap implementation omitted for brevity (See standard lib docs)
// Assuming IntHeap satisfies heap.Interface

func HeapSort(data []int) {
    h := &IntHeap{}
    *h = data
    heap.Init(h) // O(n)
    
    // Popping elements gives us sorted order
    for h.Len() > 0 {
        fmt.Printf("%d ", heap.Pop(h))
    }
}
```

### From Scratch Implementation
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
*   **Introsort**: Go's standard library uses Heap Sort as a fallback mechanism within Quick Sort (`pdqsort`). If Quick Sort's recursion goes too deep (indicating a worst-case scenario), it switches to Heap Sort to guarantee $O(n \log n)$ performance.
*   **Priority Queues**: While Heap Sort is great, Heaps are most commonly used in Go for Priority Queues (via `container/heap`) rather than for sorting entire arrays.

## Interview Questions

**Q: Why use Heap Sort over Quick Sort?**
**A:** Heap Sort guarantees $O(n \log n)$ time complexity even in the worst case, whereas Quick Sort can degrade to $O(n^2)$. It is also $O(1)$ space, unlike Merge Sort ($O(n)$).

**Q: Why is Heap Sort typically slower than Quick Sort in practice?**
**A:** **Cache Locality**. Heap Sort jumps around the array (parent to child indices $i \to 2i$) which causes frequent cache misses. Quick Sort scans arrays linearly during partitioning, which is much friendlier to modern CPU caches.

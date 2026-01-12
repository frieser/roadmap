---
---

# Heap (Binary Heap)

## Summary
A **Heap** is a specialized tree-based data structure that satisfies the **Heap Property**. It is commonly implemented as a **Priority Queue**.
*   **Max-Heap**: The parent key is always $\ge$ children keys. (Root is Max).
*   **Min-Heap**: The parent key is always $\le$ children keys. (Root is Min).

## Detailed Explanation

### Structure
Heaps are usually **Complete Binary Trees**, which allows them to be stored efficiently in an **Array** without using pointers.
*   Root at index `0`.
*   Left Child of `i`: `2*i + 1`
*   Right Child of `i`: `2*i + 2`
*   Parent of `i`: `(i-1) / 2`

### Complexity
| Operation | Time | Notes |
| :--- | :--- | :--- |
| **Get Max/Min** | $O(1)$ | Root element. |
| **Insert** | $O(\log n)$ | Add to end, then "bubble up". |
| **Delete Max/Min** | $O(\log n)$ | Swap root with end, "bubble down". |
| **Build Heap** | $O(n)$ | Using Floyd's algorithm. |

## Code Examples (Go)
Go provides `container/heap`. You must implement the `heap.Interface`.

```go
package main

import (
    "container/heap"
    "fmt"
)

// IntHeap is a min-heap of ints.
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] } // Min-Heap
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

// Push/Pop use pointer receivers because they modify the slice's length
func (h *IntHeap) Push(x any) {
    *h = append(*h, x.(int))
}

func (h *IntHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}

func main() {
    h := &IntHeap{2, 1, 5}
    heap.Init(h) // O(n)
    heap.Push(h, 3)
    fmt.Printf("Min: %d\n", (*h)[0]) // 1
    
    min := heap.Pop(h)
    fmt.Printf("Popped Min: %d\n", min)
}
```

## Interview Questions

**Q: Where are Heaps used in real systems?**
**A:**
1.  **Job Schedulers**: To pick the highest priority task efficiently.
2.  **Timers**: Managing strict timeouts (e.g., `setTimeout` in JS engines or Go runtime timers).
3.  **Graph Algorithms**: Dijkstra's Shortest Path and Prim's MST use Priority Queues.

**Q: Can you search for an element in a Heap in $O(\log n)$?**
**A:** **No.** Heaps are ordered only vertically (Parent vs Child), not horizontally. Searching requires scanning the array, which is $O(n)$.

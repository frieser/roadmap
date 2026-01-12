---
---

# Heap

## Abstract
A **Heap** is a specialized tree-based data structure that satisfies the **heap property**: in a *max heap*, for any given node `C`, if `P` is a parent node of `C`, then the key (the value) of `P` is greater than or equal to the key of `C`. In a *min heap*, the key of `P` is less than or equal to the key of `C`. The node at the "top" of the heap (with no parents) is called the **root** node. Heaps are the most efficient implementation of a **Priority Queue**.

## Development

### Core Concept
A heap is typically implemented as a **Binary Heap**, which is a **complete binary tree**. This means all levels of the tree are fully filled except possibly the last level, which is filled from left to right. This structure allows heaps to be efficiently represented as an **array**, avoiding the overhead of pointers.

#### Array Representation
For a node at index `i` (0-indexed):
- **Left Child**: `2*i + 1`
- **Right Child**: `2*i + 2`
- **Parent**: `(i - 1) / 2`

#### Operations & Complexity

| Operation | Complexity | Description |
|-----------|------------|-------------|
| **Peek**  | **O(1)**   | Access the root (min/max) element. |
| **Insert**| **O(log n)**| Add at end, then "bubble up" (sift up) to correct position. |
| **Delete**| **O(log n)**| Remove root, move last element to root, then "bubble down" (heapify down). |
| **Init**  | **O(n)**   | Build a heap from an unordered array (Floyd's algorithm). |

### Min-Heap vs Max-Heap
- **Min-Heap**: The root is the **smallest** element. Used for tasks like "process shortest job next".
- **Max-Heap**: The root is the **largest** element. Used for tasks like "process highest priority task next".

## Code Examples (Go)

Go's standard library provides the `container/heap` package. Unlike other languages that might provide a concrete class (like `PriorityQueue` in Java), Go defines a `heap.Interface` that you must implement on your own custom type.

### 1. The Interface
To use `container/heap`, your type must implement `sort.Interface` (Len, Less, Swap) plus `Push` and `Pop`.

```go
type Interface interface {
    sort.Interface
    Push(x any) // add x as element Len()
    Pop() any   // remove and return element Len() - 1.
}
```

### 2. Basic Min-Heap (Int Heap)
```go
package main

import (
    "container/heap"
    "fmt"
)

// IntHeap is a min-heap of ints.
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] } // < for Min-Heap
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

// Push and Pop use pointer receivers because they modify the slice's length,
// not just its contents.
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
    heap.Init(h) // O(n) build
    heap.Push(h, 3)
    fmt.Printf("minimum: %d\n", (*h)[0]) // 1

    for h.Len() > 0 {
        fmt.Printf("%d ", heap.Pop(h))
    }
    // Output: 1 2 3 5
}
```

### 3. Priority Queue (Struct Heap)
A more realistic example managing items with priority.

```go
// Item is something we manage in a priority queue.
type Item struct {
    value    string // The value of the item; arbitrary.
    priority int    // The priority of the item in the queue.
    index    int    // The index of the item in the heap.
}

// PriorityQueue implements heap.Interface and holds Items.
type PriorityQueue []*Item

func (pq PriorityQueue) Len() int { return len(pq) }

// We want Pop to give us the highest, not lowest, priority so we use greater than here.
func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].priority > pq[j].priority
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PriorityQueue) Push(x any) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() any {
    old := *pq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil  // avoid memory leak
    item.index = -1 // for safety
    *pq = old[0 : n-1]
    return item
}
```

## Go Application & Ecosystem

### Why an Interface?
Go's design choice to make `heap` an interface rather than a struct allows for extreme flexibility. You can heapify **any** underlying data structure that supports indexing and swapping, not just slices (though slices are 99% of use cases). It also avoids the overhead of wrapping every element in a generic node object.

### Common Use Cases
1.  **Task Scheduling**: Process jobs with highest priority first.
2.  **Graph Algorithms**: Used in Dijkstra's (shortest path) and Prim's (MST) algorithms.
3.  **K-way Merge**: Merging multiple sorted streams efficiently.
4.  **Timers**: The Go runtime uses a heap to manage sleeping goroutines and timers (`time.After`, `time.Sleep`).

## Interview Preparation

### Common Questions

1.  **What is the time complexity of building a heap from an unsorted array?**
    -   **Answer**: **O(n)**. While inserting `n` elements one by one takes O(n log n), the "heapify" algorithm (building from bottom up) takes linear time O(n).

2.  **How do you find the Kth largest element in a stream?**
    -   **Answer**: Use a **Min-Heap** of size `K`.
        1.  Add elements to the heap.
        2.  If the heap size > K, remove the minimum (root).
        3.  The root of the heap is always the Kth largest element seen so far.

3.  **Heap vs Binary Search Tree (BST)?**
    -   **Answer**:
        -   **Ordering**: BST guarantees left < root < right (sorted). Heap only guarantees parent >= children (weak ordering).
        -   **Search**: BST is O(log n) for searching arbitrary elements. Heap is O(n).
        -   **Purpose**: Use BST for searching/sorting. Use Heap for finding Min/Max efficiently.

4.  **How is a Binary Heap represented in memory?**
    -   **Answer**: Usually as an **Array** (or Slice in Go). It's a complete binary tree, so parent/child relationships are calculated via index arithmetic (`2i+1`, `2i+2`), avoiding pointer overhead.

5.  **What is the "Heap Property"?**
    -   **Answer**: In a Max-Heap, every parent node is greater than or equal to its children. In a Min-Heap, every parent is less than or equal to its children. This property must recursively hold true for all nodes.

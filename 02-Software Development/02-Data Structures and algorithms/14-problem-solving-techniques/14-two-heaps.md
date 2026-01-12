---
---

# Two Heaps

## Summary
The **Two Heaps** pattern uses two priority queues (a **Min-Heap** and a **Max-Heap**) to solve problems involving finding the **Median** of a data stream or balancing two halves of a dataset.

## Detailed Explanation

### Mechanism (Find Median)
*   **Max-Heap**: Stores the smaller half of the numbers.
*   **Min-Heap**: Stores the larger half of the numbers.
*   **Balance**: Ensure size difference is at most 1.
*   **Median**:
    *   If odd count: Top of the larger heap.
    *   If even count: Average of both tops.

## Code Examples (Go)
*Requires `container/heap` boilerplate.*

### Concept
```go
// MaxHeap (Small Half) | MinHeap (Large Half)
// [1, 2, 3]            | [4, 5]
// Median = 3
```

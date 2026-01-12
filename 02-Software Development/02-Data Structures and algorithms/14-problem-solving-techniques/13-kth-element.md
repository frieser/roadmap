---
---

# K-th Element (Top K Elements)

## Summary
This pattern is used to find the $K$-th smallest, largest, or most frequent elements in a dataset. It is efficiently solved using a **Heap (Priority Queue)** or **QuickSelect** algorithm.

## Detailed Explanation

### Approaches
1.  **Sorting**: Sort and pick index $K$. ($O(n \log n)$).
2.  **Min-Heap (for Top K Largest)**: Maintain a heap of size $K$. If new element > root, pop root and push new. ($O(n \log K)$).
3.  **QuickSelect**: Partition like QuickSort, but only recurse into the half containing $K$. ($O(n)$ avg, $O(n^2)$ worst).

## Code Examples (Go)

### K-th Largest using Min-Heap (`container/heap`)
Ideally keeps the $K$ largest elements seen so far. The smallest of the largest is at the root.

```go
// Implementation requires defining the Heap interface (See Heap notes)
// Logic:
// 1. Push elements until size K
// 2. For remaining elements: if x > heap.Peek(), heap.Pop(), heap.Push(x)
// 3. Return heap.Peek()
```

---
---

# LeetCode

## Summary
**LeetCode** is the industry standard platform for preparing for technical coding interviews. It offers a massive library of problems categorized by difficulty (Easy, Medium, Hard), topic (Array, DP, Tree), and company tags (Google, Amazon, Meta).

## Why LeetCode?
*   **Real Interview Questions**: Many companies pull questions directly from LeetCode.
*   **Community Solutions**: The "Discuss" section is a goldmine for optimizing code and learning different approaches.
*   **Contests**: Weekly contests simulate the pressure of a real timed interview.

## Recommended Strategy for Go Developers

### 1. Blind 75 / NeetCode 150
Don't solve random problems. Follow a structured list like **Blind 75** or **NeetCode 150**. These cover all major patterns (Sliding Window, Two Pointers, Trees, Graphs, DP).

### 2. Mastering Go Patterns
LeetCode supports Go natively. Use this to practice idiomatic Go:
*   **Slices**: Master `append`, slicing `arr[1:]`, and pre-allocation `make([]int, 0, n)`.
*   **Maps**: Use `map[type]struct{}` for Sets (Go doesn't have a built-in Set type until generic sets become standard).
*   **Heaps**: Get comfortable implementing `heap.Interface`. You *will* need it for "K-th Element" or "Merge K Lists" problems.

### 3. Time Management
*   **Easy**: < 15 mins.
*   **Medium**: < 30 mins.
*   **Hard**: < 45-60 mins.
If stuck, look at the solution, understand it, write it from scratch, and revisit it in 3 days.

## Key Go Snippets for LeetCode

### Priority Queue (Min-Heap) Boilerplate
Save this snippet. You'll need it often.
```go
import "container/heap"

type IntHeap []int
func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *IntHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *IntHeap) Pop() any {
    old := *h; n := len(old); x := old[n-1]; *h = old[0 : n-1]; return x
}
```

### 2D Grid Traversal (Directions Array)
```go
dirs := [][]int{{0, 1}, {0, -1}, {1, 0}, {-1, 0}}
for _, d := range dirs {
    nr, nc := r+d[0], c+d[1]
    // Check bounds
}
```

---
---

# O(n) Linear Time

## Summary
**$O(n)$ Linear Time** means the runtime scales **directly proportionally** to the input size. If you double the data, you double the time. It is the baseline efficiency for algorithms that must examine every single element of the input.

## Characteristics
*   **Scalability**: Fair/Good. Predictable growth.
*   **Mechanism**: Iterating through a collection once.
*   **Graph**: A straight diagonal line.

## Common Operations
1.  **Traversal**: Printing a list, summing an array.
2.  **Linear Search**: Finding an item in an unsorted list.
3.  **Copying**: Cloning an array or string.
4.  **Bucket Sort**: (Special case sorting).

## Go Code Examples

### 1. Simple Iteration
The most common $O(n)$ pattern in Go.

```go
// Sum is O(n)
// We visit every integer exactly once.
func Sum(nums []int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
```

### 2. Linear Search
When data is unsorted, we have no choice but to check one by one.

```go
// Contains is O(n) (Worst Case)
func Contains(nums []int, target int) bool {
    for _, n := range nums {
        if n == target {
            return true // Best case O(1), but Big O is worst case
        }
    }
    return false
}
```

### 3. String/Slice Copy
Go's built-in `copy()` function is optimized assembly, but it is physically bound by memory bandwidth, making it $O(n)$.

```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, len(src))
copy(dst, src) // O(n) - moves N integers
```

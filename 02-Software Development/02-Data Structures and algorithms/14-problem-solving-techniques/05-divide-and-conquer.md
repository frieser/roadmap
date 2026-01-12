---
---

# Divide and Conquer

## Summary
**Divide and Conquer** is a strategy where a problem is broken down into smaller sub-problems of the same type. These sub-problems are solved recursively, and their solutions are combined to form the solution to the original problem.

## Detailed Explanation

### Steps
1.  **Divide**: Break the problem into sub-problems.
2.  **Conquer**: Solve the sub-problems (recursively). Base case handles trivial inputs.
3.  **Combine**: Merge the solutions of sub-problems.

### Classic Examples
*   Merge Sort
*   Quick Sort
*   Binary Search (Decrease and Conquer)

## Code Examples (Go)

### Merge Sort (Conceptual)
```go
func MergeSort(arr []int) []int {
    if len(arr) <= 1 { return arr }
    
    // Divide
    mid := len(arr) / 2
    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])
    
    // Combine
    return Merge(left, right)
}
```

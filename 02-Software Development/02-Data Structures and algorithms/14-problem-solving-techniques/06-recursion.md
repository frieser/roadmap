---
---

# Recursion

## Summary
**Recursion** is a programming technique where a function calls itself to solve a smaller instance of the problem. It is the implementation vehicle for DFS, Divide & Conquer, and Backtracking.

## Detailed Explanation

### Anatomy of Recursion
1.  **Base Case**: The stopping condition (e.g., `if n == 0 return`). Without this, you get a Stack Overflow.
2.  **Recursive Step**: Calling the function with modified arguments moving towards the base case.

### Complexity
*   **Time**: $O(\text{branches}^{\text{depth}})$.
*   **Space**: $O(\text{depth})$ (Call Stack).

## Code Examples (Go)

### Factorial
```go
func Factorial(n int) int {
    // Base Case
    if n <= 1 {
        return 1
    }
    // Recursive Step
    return n * Factorial(n-1)
}
```

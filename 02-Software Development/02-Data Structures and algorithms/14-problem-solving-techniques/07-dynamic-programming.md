---
---

# Dynamic Programming (DP)

## Summary
**Dynamic Programming** is an optimization technique for solving recursive problems that have **Overlapping Subproblems**. Instead of solving the same subproblem multiple times, DP stores the result (Memoization/Tabulation) and reuses it.

## Detailed Explanation

### Approaches
1.  **Top-Down (Memoization)**: Recursion + Caching. Easy to write if you know the recursive formula.
2.  **Bottom-Up (Tabulation)**: Iteration. Fills a table (array/matrix) starting from the base case. Often saves stack space.

### Use Cases
*   Fibonacci
*   Knapsack Problem
*   Longest Common Subsequence

## Code Examples (Go)

### Fibonacci (Memoization)
```go
func FibMemo(n int, memo map[int]int) int {
    if n <= 1 { return n }
    if val, ok := memo[n]; ok { return val }
    
    memo[n] = FibMemo(n-1, memo) + FibMemo(n-2, memo)
    return memo[n]
}
```

### Fibonacci (Tabulation)
```go
func FibTab(n int) int {
    if n <= 1 { return n }
    dp := make([]int, n+1)
    dp[0], dp[1] = 0, 1
    
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}
```

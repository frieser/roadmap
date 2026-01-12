---
---

# How to Calculate Complexity

## Summary
Calculating complexity involves analyzing the growth of an algorithm's operations and memory usage as the input $N$ scales. By following standardized rules—such as dropping constants and focusing on dominant terms—we can distill complex code into a clear Big O classification.

## Detailed Explanation

### 1. Calculation Rules

#### Rule 1: Drop Constants
Big O notation focuses on the **growth rate**, not the exact number of operations. $O(2N)$ becomes $O(N)$ and $O(500)$ becomes $O(1)$.
```go
func rule1(n int) {
    // Two separate loops
    for i := 0; i < n; i++ { /* ... */ } // O(n)
    for i := 0; i < n; i++ { /* ... */ } // O(n)
    // Total: O(2n) -> O(n)
}
```

#### Rule 2: Dominant Terms
Keep the highest power of $N$ and discard the rest. $O(N^2 + N)$ simplifies to $O(N^2)$.
```go
func rule2(n int) {
    // Nested loop: O(n^2)
    for i := 0; i < n; i++ {
        for j := 0; j < n; j++ { /* ... */ }
    }
    // Single loop: O(n)
    for i := 0; i < n; i++ { /* ... */ }
    // O(n^2 + n) -> O(n^2)
}
```

#### Rule 3: Loops (Multiply)
Inside a loop, operations are multiplied by the number of iterations. Nested loops lead to $O(N^2)$, $O(N^3)$, etc.

#### Rule 4: Recursion (Tree Depth)
For recursion, complexity is often $O(\text{Branches}^{\text{Depth}})$.
- **Factorial**: $O(N!)$
- **Fibonacci (Naive)**: $O(2^N)$

### 2. Go Specifics

#### Recursion Depth Limits
Unlike some languages with large default stacks, Go goroutines start with a very small stack (2KB) that grows. However, deep recursion can still lead to "Stack Overflow" if memory is exhausted or if the runtime limits are reached.
- **Tail Call Optimization**: Go does **not** currently implement tail-call optimization. Recursive functions will always increase the stack depth.

#### Map and Slice Iteration
Iterating over a `map` or `slice` of size $N$ is always $O(N)$. However, map lookups are $O(1)$ on average.

## Complexity Analysis Example (Go)

```go
func findDuplicates(nums []int) []int {
    seen := make(map[int]bool) // Space: O(n)
    result := []int{}

    for _, n := range nums { // Time: O(n)
        if seen[n] {
            result = append(result, n)
        } else {
            seen[n] = true
        }
    }
    return result
}
// Overall: Time O(n), Space O(n)
```

## Interview Questions
1. **Q: What is the complexity of binary search?**
   - **A:** $O(\log N)$, because the input size is halved at each step.
2. **Q: How do you calculate the complexity of a nested loop where the inner loop depends on the outer loop (e.g., j < i)?**
   - **A:** It is still $O(N^2)$. The total iterations are $1 + 2 + ... + N = \frac{N(N+1)}{2}$, which simplifies to $O(N^2)$ after dropping constants.
3. **Q: Why is naive Fibonacci recursion $O(2^N)$?**
   - **A:** Because each call generates two more calls, creating a binary tree of height $N$. The number of nodes in a full binary tree is $2^N - 1$.

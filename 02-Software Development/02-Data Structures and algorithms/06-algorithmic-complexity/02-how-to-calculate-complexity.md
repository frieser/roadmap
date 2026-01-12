---
---

# How to Calculate Complexity

## Summary
Calculating complexity involves analyzing the code to derive a mathematical function $f(N)$ that describes resource usage, then simplifying it to its asymptotic bound (Big O). The process focuses on **Worst-Case** scenarios and ignores constants, hardware differences, and non-dominant terms.

## Detailed Explanation

### 1. The Rules of Simplification
To convert an exact instruction count (e.g., $3N^2 + 50N + 100$) to Big O:

1.  **Drop Constants**: $2N \to O(N)$, $500 \to O(1)$. Hardware speed varies; scaling factor does not.
2.  **Drop Non-Dominant Terms**: $N^2 + N \to O(N^2)$. As $N \to \infty$, the largest exponent dwarfs everything else.
3.  **Worst Case Assumption**: Unless specified ("Average Case"), assume the condition that forces the most work (e.g., searching for an item at the very end of a list).

### 2. Calculating Time Complexity

#### Sequential Statements (Addition)
Add the complexity of code blocks executed one after another.
```go
func process(n int) {
    // Block A: O(N)
    for i := 0; i < n; i++ { ... }

    // Block B: O(1)
    fmt.Println("Done")
}
// Total: O(N) + O(1) = O(N)
```

#### Loops (Multiplication)
A loop running $N$ times multiplies the complexity of its body.
*   **Simple Loop**: $N \times O(1) = O(N)$.
*   **Nested Loop**: $N \times N = O(N^2)$.
*   **Independently Nested**: $N \times M = O(N \times M)$ (Common in matrix ops).

#### Logarithmic Loops
If the loop variable is multiplied/divided (e.g., `i *= 2`), the complexity is $O(\log N)$.
```go
for i := 1; i < n; i *= 2 {
    // Runs 1, 2, 4, 8...
    // Total steps: log2(n)
}
```

### 3. Calculating Recursive Complexity
Recursion is harder to visualize. Use the "Branching" formula:

$$ O(\text{Branches}^{\text{Depth}}) $$

*   **Branches**: How many recursive calls per function?
*   **Depth**: How many layers until the base case?

**Example: Fibonacci**
*   Calls itself 2 times (Branches = 2).
*   Goes down to 0 (Depth = N).
*   Complexity: $O(2^N)$.

**Example: Binary Search**
*   Calls itself 1 time (Branches = 1).
*   Halves the input (Depth = $\log N$).
*   Complexity: $O(\log N)$.

## Go Specifics

### `append()` Amortized Analysis
Appending to a slice is tricky.
*   Most calls are $O(1)$ (writing to empty space).
*   Ideally every $N$ calls, we resize ($O(N)$ copy).
*   **Amortized Complexity**: Averaged over time, it is **$O(1)$**. Do not treat `append` inside a loop as $O(N^2)$ unless you are specifically analyzing worst-case latency spikes.

### Built-in Functions
Know the cost of Go's built-ins:
*   `copy(dst, src)`: $O(N)$.
*   `len(slice)`: $O(1)$ (It reads a header field, doesn't count).
*   `map[key]`: $O(1)$ usually, $O(N)$ worst-case collisions.

## Interview Questions

**Q: What is the complexity of this code?**
```go
for i := 0; i < N; i++ {
    for j := 0; j < i; j++ {
        print(j)
    }
}
```
**A:** **$O(N^2)$**. Even though the inner loop runs `i` times (not `N`), the sum of steps is $1 + 2 + ... + N = N(N+1)/2$, which simplifies to $O(N^2)$.

**Q: Why do we drop constants like "2N"?**
**A:** Because Big O describes the **rate of growth**, not the exact speed. An algorithm taking $2N$ seconds grows linearly, just like $N$. On a machine 2x faster, $2N$ becomes $N$. The curve shape is identical.

**Q: How do you handle multiple variables, like searching for a string of length S in an array of N strings?**
**A:** Express it as **$O(N \times S)$**. You cannot drop either variable unless you know a relationship between them (e.g., if $S$ is always constant, it's $O(N)$).

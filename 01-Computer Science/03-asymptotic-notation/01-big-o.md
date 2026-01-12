---
---

# Big O Notation (O)

## Abstract
**Big O Notation** ($O$) describes the **upper bound** of an algorithm's growth rate. It answers the question: "In the worst-case scenario, how does the runtime or space usage grow as the input size ($n$) increases?" It is the industry standard for discussing algorithmic efficiency because it guarantees that the algorithm will **never** perform worse than this limit.

## Development

### Core Concept
Mathematically, $f(n) = O(g(n))$ if there exist positive constants $c$ and $n_0$ such that:
$$ 0 \le f(n) \le c \cdot g(n) \quad \text{for all } n \ge n_0 $$

This means that for large inputs, the function $f(n)$ (actual runtime) is always less than or equal to some multiple of $g(n)$ (the notation).

### Why Big O?
Engineers care about the **Worst Case** because:
- It provides a **safety guarantee**.
- It helps prevent system outages under heavy load.
- Malicious users often exploit worst-case scenarios (e.g., HashDoS attacks).

### Example: Linear Search
Searching for an item in an unsorted slice.
- **Best Case**: Item is first ($O(1)$).
- **Worst Case**: Item is last or not present ($O(n)$).
- **Big O**: $O(n)$ (Because we care about the upper bound).

## Code Examples (Go)

### 1. O(n) - Linear Scan
Even if the loop breaks early, the Big O is determined by the worst-case possibility.

```go
func contains(nums []int, target int) bool {
    for _, n := range nums {
        if n == target {
            return true // Might return here (Best case)
        }
    }
    return false // Worst case: iterated everything
}
// Complexity: O(n)
```

### 2. O(n^2) - Worst Case Limit
Nested loops define the upper bound.

```go
func printPairs(nums []int) {
    n := len(nums)
    for i := 0; i < n; i++ {
        for j := 0; j < n; j++ {
            fmt.Println(nums[i], nums[j])
        }
    }
}
// Complexity: O(n^2)
```

## Go Application & Ecosystem
- **Standard Library**: Documentation often implies Big O. `sort.Strings` is $O(n \log n)$.
- **Timeouts**: Understanding Big O is crucial when setting `context.WithTimeout`. An $O(n^2)$ function might work in tests ($n=10$) but timeout in production ($n=10,000$).

## Interview Preparation
1.  **Does O(n) mean exactly n steps?**
    -   *Answer*: No. It means the steps grow *linearly* with $n$. It could be $10n$ or $0.5n$ steps. Constants are dropped.
2.  **Why do we drop constants?**
    -   *Answer*: Because as $n \to \infty$, constants become insignificant compared to the growth factor. $O(2n)$ and $O(n)$ describe the same linear trend.
3.  **Can Big O be less than the worst case?**
    -   *Answer*: Technically yes ($n = O(n^2)$ is true), but in CS we usually mean the **tightest** upper bound.

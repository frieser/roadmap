---
---

# Big Omega Notation (Ω)

## Abstract
**Big Omega Notation** ($\Omega$) describes the **lower bound** of an algorithm. It answers: "What is the absolute **best** performance we can hope for?" While Big O gives the safety ceiling, Big Omega gives the floor. It is less commonly used in engineering trade-offs but critical for proving algorithmic optimality (e.g., proving that sorting cannot be faster than $\Omega(n \log n)$).

## Development

### Core Concept
Mathematically, $f(n) = \Omega(g(n))$ if there exist positive constants $c, n_0$ such that:
$$ 0 \le c \cdot g(n) \le f(n) \quad \text{for all } n \ge n_0 $$

This means the algorithm will take **at least** this amount of time.

### Context
- **Linear Search**: Best case is finding the element at index 0.
    - $\Omega(1)$ (Best case).
    - $O(n)$ (Worst case).
- **Sorting**: Comparison-based sorting (Merge, Heap, Quick) requires at least $\Omega(n \log n)$ comparisons in the worst case to distinguish all permutations.

## Code Examples (Go)

### 1. Early Exit Loop
This function is $O(n)$ but $\Omega(1)$.

```go
func hasEven(nums []int) bool {
    for _, n := range nums {
        if n%2 == 0 {
            return true // Returns immediately (Best case Ω(1))
        }
    }
    return false
}
```

### 2. Printing a Matrix
This function is $\Omega(n^2)$ because we MUST visit every cell. We cannot do it faster.

```go
func printMatrix(mat [][]int) {
    for _, row := range mat {
        for _, val := range row {
            fmt.Print(val)
        }
    }
}
// Lower bound is n^2. You cannot print n^2 items in less than n^2 time.
```

## Go Application
- **Optimizations**: Knowing the $\Omega$ bound helps you know when to stop optimizing. If a problem requires reading $n$ items, you cannot solve it in less than $\Omega(n)$.
- **Short-circuiting**: Go's `&&` and `||` operators provide $\Omega(1)$ behavior in boolean logic (if left side determines result, right is skipped).

## Interview Preparation
1.  **What is the lower bound of sorting?**
    -   *Answer*: $\Omega(n \log n)$ for comparison-based sorts. $\Omega(n)$ for non-comparison sorts (Radix sort).
2.  **Is "Best Case" synonymous with Big Omega?**
    -   *Answer*: Roughly, yes. While $\Omega$ mathematically describes a lower bound for *any* case, in interviews it's often used to describe the best-case runtime.
3.  **Can an algorithm be O(n^2) and Ω(n)?**
    -   *Answer*: Yes. Insertion Sort is $O(n^2)$ (worst case) and $\Omega(n)$ (best case, already sorted).

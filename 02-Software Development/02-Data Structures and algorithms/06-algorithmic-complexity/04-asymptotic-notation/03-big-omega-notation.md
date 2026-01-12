---
---

# Big Omega Notation ($\Omega$)

## Summary
**Big Omega Notation ($\Omega$)** defines the **Lower Bound** of an algorithm. It answers the question: *"What is the best-case scenario?"* or *"It will take at least this much time."* While less critical for safety than Big O, it is useful for proving that an algorithm cannot potentially get any faster.

## Formal Definition
$f(n) = \Omega(g(n))$ if there exist constants $c > 0$ and $n_0 \ge 0$ such that:
$$ f(n) \ge c \cdot g(n) \quad \text{for all } n \ge n_0 $$

In plain English: The algorithm takes **at least** this amount of time.

## Why It Matters
*   **Theoretical Limits**: We know that comparison-based sorting cannot be faster than $\Omega(N \log N)$. This stops us from trying to invent an impossible $O(N)$ generic sort.
*   **Early Exits**: An algorithm with a good best-case ($\Omega(1)$) works well on specific datasets (e.g., Insertion Sort on nearly-sorted data).

## Example
**Linear Search**:
*   **Best Case ($\Omega$)**: The item is at index 0. -> **$\Omega(1)$**.
*   Worst Case ($O$): The item is at index N. -> $O(N)$.

```go
// Best case: Target is arr[0]. Runtime is Constant.
// Worst case: Target is missing. Runtime is Linear.
func Find(arr []int, target int) int {
    if arr[0] == target {
        return 0
    }
    // ... rest of loop
}
```

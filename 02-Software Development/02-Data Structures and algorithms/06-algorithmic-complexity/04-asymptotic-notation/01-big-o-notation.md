---
---

# Big O Notation (O)

## Summary
**Big O Notation ($O$)** defines the **Upper Bound** of an algorithm's runtime. It answers the question: *"What is the worst-case scenario?"* or *"It will not run slower than this."* In software engineering, this is the primary metric because it guarantees system stability under load.

## Formal Definition
$f(n) = O(g(n))$ if there exist constants $c > 0$ and $n_0 \ge 0$ such that:
$$ f(n) \le c \cdot g(n) \quad \text{for all } n \ge n_0 $$

In plain English: For large inputs ($n$), the algorithm's time $f(n)$ is always below or equal to a scaled version of the curve $g(n)$.

## Why It Matters
*   **Safety**: If an algorithm is $O(N)$, we know that 10x data will result in roughly 10x time. If it were $O(N^2)$, 10x data means 100x time (potential crash).
*   **Pessimism**: Big O assumes the worst luck (e.g., the item you search for is at the very last index).

## Example
**Linear Search**:
*   Best Case: Item is first ($1$ step).
*   Average Case: Item is in middle ($N/2$ steps).
*   **Worst Case (Big O)**: Item is last ($N$ steps). -> **$O(N)$**

```go
func Contains(arr []int, target int) bool {
    for _, v := range arr {
        if v == target {
            return true
        }
    }
    return false // Worst case runs N times
}
```

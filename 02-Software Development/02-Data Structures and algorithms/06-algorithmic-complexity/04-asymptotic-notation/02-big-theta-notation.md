---
---

# Big Theta Notation ($\Theta$)

## Summary
**Big Theta Notation ($\Theta$)** defines the **Tight Bound** of an algorithm. It applies when the algorithm's Best Case and Worst Case scale in the same way. It is a more precise statement than Big O: instead of saying *"It's no worse than X"*, it says *"It behaves exactly like X"*.

## Formal Definition
$f(n) = \Theta(g(n))$ if $f(n)$ is bounded both above **and** below by $g(n)$.
$$ c_1 \cdot g(n) \le f(n) \le c_2 \cdot g(n) $$

## Why It Matters
*   **Precision**: Big O can be loose. Technically, a linear scan $O(N)$ is also $O(N^2)$ (because $N < N^2$), but that's not helpful. $\Theta(N)$ tells you specifically that it is linear.
*   **Consistency**: Algorithms like Merge Sort are $\Theta(N \log N)$ because they take the same amount of time regardless of whether the input is sorted or random.

## Example
**Iterating an Array**:
You must visit every element to sum them.
*   Best Case: $N$ steps.
*   Worst Case: $N$ steps.
*   Result: **$\Theta(N)$**.

```go
func Sum(arr []int) int {
    s := 0
    for _, v := range arr {
        s += v // Always runs N times
    }
    return s
}
```

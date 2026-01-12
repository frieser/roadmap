---
---

# Big Theta Notation (Θ)

## Abstract
**Big Theta Notation** ($\Theta$) describes the **tight bound** of an algorithm. It brackets the runtime from both above and below. When we say an algorithm is $\Theta(n)$, it means the runtime is **always** proportional to $n$, regardless of the input configuration (best or worst case). It provides a more precise and stronger claim than Big O.

## Development

### Core Concept
Mathematically, $f(n) = \Theta(g(n))$ if there exist positive constants $c_1, c_2, n_0$ such that:
$$ 0 \le c_1 \cdot g(n) \le f(n) \le c_2 \cdot g(n) \quad \text{for all } n \ge n_0 $$

In simpler terms: The algorithm runs in time bounded by $k_1 \cdot n$ and $k_2 \cdot n$. It doesn't fluctuate wildly between best and worst cases.

### Big O vs Big Theta
- **Big O**: "It won't be worse than this." (e.g., Linear Search is $O(n)$).
- **Big Theta**: "It behaves exactly like this." (e.g., Linear Search is **NOT** $\Theta(n)$ because it could be $O(1)$ if the item is first).

### When does f(n) = Θ(g(n))?
Only if $f(n) = O(g(n))$ **AND** $f(n) = \Omega(g(n))$.
The Upper Bound and Lower Bound must match.

## Code Examples (Go)

### 1. Traversing an Array (True Θ(n))
Printing every element requires visiting $n$ items no matter what. Best case = Worst case = $n$.

```go
func printAll(nums []int) {
    for _, n := range nums {
        fmt.Println(n)
    }
}
// Best case: n steps
// Worst case: n steps
// Conclusion: Θ(n)
```

### 2. Merge Sort (True Θ(n log n))
Merge Sort always divides the array and merges it back, regardless of whether the input is sorted or not.

```go
// Merge Sort structure
func mergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    mid := len(arr) / 2
    left := mergeSort(arr[:mid])
    right := mergeSort(arr[mid:])
    return merge(left, right)
}
// Always does the same split/merge work.
// Conclusion: Θ(n log n)
```

## Go Application
- **Fixed Overhead**: Functions that always do a fixed amount of work per element (like `copy` or `len` calculation loops) are $\Theta(n)$.
- **Predictability**: Algorithms with $\Theta$ complexity are preferred in real-time systems where jitter (variance in execution time) is undesirable.

## Interview Preparation
1.  **Is Linear Search Θ(n)?**
    -   *Answer*: **No**. Because its best case is $O(1)$ and worst is $O(n)$. Since $1 \neq n$, it has no tight bound $\Theta(n)$.
2.  **Is Insertion Sort Θ(n^2)?**
    -   *Answer*: **No**. On almost sorted data, it runs in $O(n)$. In worst case (reversed), it is $O(n^2)$.
3.  **Why do people say "O(n)" when they mean "Θ(n)"?**
    -   *Answer*: It's a common industry colloquialism. Strictly speaking, saying "Iterating an array is O(n)" is correct, but "Θ(n)" is more precise.

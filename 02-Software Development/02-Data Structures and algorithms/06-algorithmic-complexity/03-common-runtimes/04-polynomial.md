---
---

# O(n^2) Polynomial Time

## Summary
**$O(n^2)$ Quadratic Time** is the most common form of "Polynomial Time". It occurs when the runtime grows proportionally to the **square** of the input. It typically signals a "brute force" approach involving nested iterations over the same data.

## Characteristics
*   **Scalability**: Poor. Doubling the input quadruples the runtime ($2x \to 4x$).
    *   $N=10 \to 100$ steps.
    *   $N=1000 \to 1,000,000$ steps (1 million!).
*   **Mechanism**: Nested loops ($N \times N$).
*   **Graph**: A parabola (J-curve) that shoots up rapidly.

## Common Operations
1.  **Nested Loops**: Iterating a matrix, comparing every pair.
2.  **Basic Sorts**: Bubble Sort, Insertion Sort, Selection Sort.
3.  **Dup Check (Naive)**: Checking for duplicates by scanning the rest of the list for each item.

## Go Code Examples

### 1. Nested Loops (Bubble Sort)
The classic example of $O(n^2)$.

```go
// BubbleSort is O(n^2)
func BubbleSort(arr []int) {
    n := len(arr)
    // Outer loop: N times
    for i := 0; i < n; i++ {
        // Inner loop: Roughly N times
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                // Swap
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}
```

### 2. Pairwise Comparison
Comparing every element with every other element.

```go
// FindDuplicates is O(n^2)
// Slow! Use a Map (O(n)) instead for production.
func FindDuplicates(arr []int) {
    for i := 0; i < len(arr); i++ {
        for j := i + 1; j < len(arr); j++ {
            if arr[i] == arr[j] {
                fmt.Printf("Duplicate: %d\n", arr[i])
            }
        }
    }
}
```

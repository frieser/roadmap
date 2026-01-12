---
---

# Polynomial Time - O(n^2), O(n^3)

## Abstract
**Polynomial Time** refers to algorithms where the runtime grows based on $n$ raised to a power, typically **Quadratic O(n²)** or **Cubic O(n³)**. These runtimes usually indicate **nested loops**. While acceptable for small inputs, they scale poorly and often cause performance bottlenecks in production systems with large datasets.

## Development

### Quadratic - O(n²)
- **Growth**: If $n$ doubles, time quadruples ($2^2 = 4$).
- **Source**: Nested loops (loop inside a loop).
- **Examples**: Bubble Sort, Insertion Sort, checking all pairs of items.

### Cubic - O(n³)
- **Growth**: If $n$ doubles, time increases by $8x$ ($2^3 = 8$).
- **Source**: Triple nested loops.
- **Examples**: Matrix multiplication (naive), some dynamic programming solutions.

### The Danger Zone
For $n = 100,000$:
- O(n) $\approx 0.1$ ms
- O(n²) $\approx 10,000$ ms (10 seconds)
- O(n³) $\approx$ Years.

## Code Examples (Go)

### 1. Quadratic (O(n²)) - All Pairs
Printing every possible pair of elements.
```go
func printPairs(nums []int) {
    n := len(nums)
    // Outer loop runs n times
    for i := 0; i < n; i++ {
        // Inner loop runs n times for every outer iteration
        for j := 0; j < n; j++ {
            fmt.Println(nums[i], nums[j])
        }
    }
}
// Total operations: n * n = n^2
```

### 2. Bubble Sort (O(n²))
A classic inefficient sorting algorithm.
```go
func bubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}
```

### 3. Cubic (O(n³)) - Naive Matrix Multiplication
```go
func multiplyMatrices(a, b [][]int) [][]int {
    n := len(a)
    result := make([][]int, n)
    for i := 0; i < n; i++ {
        result[i] = make([]int, n)
        for j := 0; j < n; j++ {
            for k := 0; k < n; k++ {
                result[i][j] += a[i][k] * b[k][j]
            }
        }
    }
    return result
}
```

## Go Application & Ecosystem
- **Accidental O(n²)**: A common Go pitfall is using `append` inside a loop where the slice capacity is not pre-allocated, or doing string concatenation `s += "str"` inside a large loop (though modern Go optimizes this, `strings.Builder` is preferred).
- **LeetCode/Interview**: Often you start with an O(n²) brute force solution and are asked to optimize it to O(n) or O(n log n).

## Interview Preparation
1.  **When is O(n²) acceptable?**
    -   *Answer*: When $n$ is small (e.g., < 1000) or when no faster algorithm exists (e.g., some graph problems).
2.  **How to spot O(n²) quickly?**
    -   *Answer*: Look for nested loops where both iterate up to $n$.
3.  **Is `append` inside a loop O(n²)?**
    -   *Answer*: It can be. If you repeatedly append to a slice that causes frequent re-allocations and full copies, the total cost can degrade.

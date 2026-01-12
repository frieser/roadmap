---
---

# O(2^n) Exponential Time

## Summary
**$O(2^n)$ Exponential Time** describes an algorithm whose growth rate **doubles** with each additional element in the input. This is considered **intractable** for large datasets. Algorithms with this complexity are usually limited to very small inputs ($N < 40$).

## Characteristics
*   **Scalability**: Horrible. Adding 1 item doubles the work. Adding 10 items multiplies work by 1024.
*   **Mechanism**: Recursive functions that make 2 (or more) calls per step without reusing results.
*   **Graph**: A "hockey stick" that goes vertical almost immediately.

## Common Operations
1.  **Recursive Fibonacci (Naive)**.
2.  **Power Set**: Generating all subsets of a set.
3.  **Brute Force Solutions**: Traveling Salesman (specific variants), Knapsack problem (naive).

## Go Code Examples

### Naive Fibonacci
The textbook example of $O(2^n)$. To calculate `Fib(5)`, it calculates `Fib(4)` and `Fib(3)`. `Fib(4)` *also* calculates `Fib(3)`... resulting in massive redundant work.

```go
package main

// Fib is O(2^n)
// N=40 takes seconds. N=50 takes minutes/hours.
func Fib(n int) int {
    if n <= 1 {
        return n
    }
    // Branching factor of 2, Depth of N
    return Fib(n-1) + Fib(n-2)
}
```

### Power Set
Generating all possible combinations of a slice.

```go
// PowerSet generates 2^N subsets
// Input: [1, 2] -> [], [1], [2], [1, 2]
func PowerSet(set []int) [][]int {
    powerSetSize := 1 << len(set) // 2^N
    result := make([][]int, 0, powerSetSize)

    for i := 0; i < powerSetSize; i++ {
        var subset []int
        for j, elem := range set {
            // Check if j-th bit is set in i
            if (i>>j)&1 == 1 {
                subset = append(subset, elem)
            }
        }
        result = append(result, subset)
    }
    return result
}
```

---
---

# Fenwick Trees (Binary Indexed Trees)

## Summary
A **Fenwick Tree** (or Binary Indexed Tree - BIT) is a data structure that provides efficient methods for calculation and manipulation of **prefix sums** of a table of values. It is space-efficient ($O(n)$) and supports both Point Update and Prefix Query in **$O(\log n)$**.

## Detailed Explanation

### Core Concept
Unlike a Segment Tree which stores explicit ranges, a Fenwick Tree uses the **binary representation** of indices to determine which ranges a node covers.
*   Index `i` is responsible for a range of length determined by the **Least Significant Bit (LSB)** of `i`.
*   Example: Index 12 (`1100`) covers range of length 4. Index 13 (`1101`) covers length 1.

### Use Case
Calculating cumulative frequencies. Ideally suited for finding "Sum of `arr[0...i]`" and updating `arr[k]`.

## Complexity
| Operation | Time | Space |
| :--- | :--- | :--- |
| **Build** | $O(n)$ | $O(n)$ |
| **Update** | $O(\log n)$ | - |
| **Prefix Sum** | $O(\log n)$ | - |

## Code Examples (Go)

```go
package main

type FenwickTree struct {
    tree []int
}

func NewFenwickTree(size int) *FenwickTree {
    return &FenwickTree{tree: make([]int, size+1)}
}

// Add delta to element at index i (1-based)
func (ft *FenwickTree) Add(i, delta int) {
    for i < len(ft.tree) {
        ft.tree[i] += delta
        i += i & (-i) // Add LSB
    }
}

// Query prefix sum up to index i (1-based)
func (ft *FenwickTree) Query(i int) int {
    sum := 0
    for i > 0 {
        sum += ft.tree[i]
        i -= i & (-i) // Subtract LSB
    }
    return sum
}
```

## Interview Questions

**Q: Why use a Fenwick Tree over a Segment Tree?**
**A:**
1.  **Space**: Fenwick Tree uses exactly $N$ integers. Segment Tree uses $4N$.
2.  **Implementation**: Fenwick Tree is ~10 lines of code using bitwise operations (`i & -i`).
3.  **Speed**: Faster constant factors due to better cache locality and bitwise ops.

**Q: Can Fenwick Trees calculate Range Minimum Query (RMQ)?**
**A:** Not easily. Fenwick Trees rely on the operation being **invertible** (like `Sum`: `Range(L, R) = Prefix(R) - Prefix(L-1)`). `Min/Max` are not invertible operations (you can't subtract a "min").

---
---

# Segment Trees

## Summary
A **Segment Tree** is a versatile tree data structure used for storing information about **intervals** (or segments). It allows for answering **range queries** (e.g., "sum of array between index $L$ and $R$") and **point/range updates** both in **$O(\log n)$** time.

## Detailed Explanation

### Structure
*   It is a (mostly) **Complete Binary Tree**.
*   **Leaves**: Represent individual array elements `arr[i]`.
*   **Internal Nodes**: Represent the aggregation (Sum, Min, Max, GCD) of their children's ranges.
    *   Example: Node covering `[0, 3]` stores sum of children covering `[0, 1]` and `[2, 3]`.

### Why not use simple Prefix Sums?
Prefix sums allow $O(1)$ range queries but require $O(N)$ to update a value (recomputing the array). Segment Trees support both **Query** and **Update** in $O(\log n)$.

## Complexity
| Operation | Time | Space |
| :--- | :--- | :--- |
| **Build** | $O(n)$ | $O(4n)$ |
| **Query (Range)** | $O(\log n)$ | - |
| **Update (Point)** | $O(\log n)$ | - |

## Code Examples (Go)
*Implementation of Range Sum Query (RSQ).*

```go
package main

type SegmentTree struct {
    tree []int
    n    int
}

func NewSegmentTree(arr []int) *SegmentTree {
    n := len(arr)
    st := &SegmentTree{
        tree: make([]int, 4*n),
        n:    n,
    }
    st.build(arr, 1, 0, n-1)
    return st
}

func (st *SegmentTree) build(arr []int, node, start, end int) {
    if start == end {
        st.tree[node] = arr[start]
    } else {
        mid := (start + end) / 2
        st.build(arr, 2*node, start, mid)
        st.build(arr, 2*node+1, mid+1, end)
        st.tree[node] = st.tree[2*node] + st.tree[2*node+1]
    }
}

// Query returns sum in range [L, R]
func (st *SegmentTree) Query(L, R int) int {
    return st.query(1, 0, st.n-1, L, R)
}

func (st *SegmentTree) query(node, start, end, L, R int) int {
    if R < start || end < L {
        return 0 // Out of range
    }
    if L <= start && end <= R {
        return st.tree[node] // Fully inside
    }
    mid := (start + end) / 2
    return st.query(2*node, start, mid, L, R) + st.query(2*node+1, mid+1, end, L, R)
}
```

## Interview Questions

**Q: What is "Lazy Propagation"?**
**A:** An optimization for **Range Updates**. Instead of updating every leaf node in a range (which takes $O(n)$), we update the high-level node and flag it as "lazy". The change is pushed down to children only when those children are explicitly accessed later. This keeps range updates at $O(\log n)$.

**Q: Segment Tree vs Fenwick Tree?**
**A:**
*   **Fenwick Tree**: Less memory ($O(n)$ vs $O(4n)$), easier to code, bitwise magic. Only supports cumulative/invertible operations (Sum, XOR).
*   **Segment Tree**: More flexible. Supports Min/Max (non-invertible) and complex range queries.

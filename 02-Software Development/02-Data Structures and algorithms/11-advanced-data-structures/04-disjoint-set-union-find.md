---
---

# Disjoint Set Union (Union-Find)

## Summary
**Disjoint Set Union (DSU)**, also known as **Union-Find**, is a data structure that tracks a set of elements partitioned into a number of disjoint (non-overlapping) subsets. It is incredibly efficient at checking whether two elements belong to the same group and merging two groups.

## Detailed Explanation

### Operations
1.  **MakeSet(x)**: Create a new set with element `x`.
2.  **Union(x, y)**: Merge the set containing `x` and the set containing `y`.
3.  **Find(x)**: Return the representative (root) of the set containing `x`.

### Optimizations (Crucial)
1.  **Path Compression**: During `Find(x)`, make all nodes on the path point directly to the root. This flattens the tree.
2.  **Union by Rank/Size**: Always attach the smaller tree to the larger tree during `Union`. This prevents deep trees.

## Complexity
With both optimizations, the amortized time complexity is **$O(\alpha(n))$**, where $\alpha$ is the **Inverse Ackermann Function**.
*   For all practical values of $n$ (even the number of atoms in the universe), $\alpha(n) \le 4$.
*   Effectively **$O(1)$** constant time.

## Code Examples (Go)

```go
package main

type DSU struct {
    parent []int
    rank   []int
}

func NewDSU(n int) *DSU {
    dsu := &DSU{
        parent: make([]int, n),
        rank:   make([]int, n),
    }
    for i := 0; i < n; i++ {
        dsu.parent[i] = i // Each node is its own parent initially
    }
    return dsu
}

func (d *DSU) Find(x int) int {
    if d.parent[x] != x {
        // Path Compression
        d.parent[x] = d.Find(d.parent[x])
    }
    return d.parent[x]
}

func (d *DSU) Union(x, y int) {
    rootX := d.Find(x)
    rootY := d.Find(y)
    
    if rootX != rootY {
        // Union by Rank
        if d.rank[rootX] < d.rank[rootY] {
            d.parent[rootX] = rootY
        } else if d.rank[rootX] > d.rank[rootY] {
            d.parent[rootY] = rootX
        } else {
            d.parent[rootY] = rootX
            d.rank[rootX]++
        }
    }
}
```

## Go Application
*   **Kruskal's Algorithm**: Finding Minimum Spanning Tree (MST).
*   **Connected Components**: Counting islands in a grid or clusters in a graph.
*   **Image Processing**: Labeling connected regions of pixels.

## Interview Questions

**Q: What is the Inverse Ackermann Function?**
**A:** It is a function that grows incredibly slowly. $\alpha(n) = 4$ for $n = 2^{2^{2^{2^{16}}}}$. In practical computer science terms, it is considered constant time.

**Q: Can DSU support "un-union" (splitting sets)?**
**A:** No. Standard DSU only supports merging. Once sets are merged, the history is lost (due to path compression). To support rollback/splitting, you need a "Persistent DSU" or "DSU with Rollback" (stack-based, no path compression).

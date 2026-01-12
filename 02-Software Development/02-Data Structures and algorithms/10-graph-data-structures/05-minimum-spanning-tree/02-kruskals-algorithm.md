---
---

# Kruskal's Algorithm

## Summary
**Kruskal's Algorithm** finds the **Minimum Spanning Tree (MST)** by sorting all edges by weight and adding them one by one, provided they don't form a cycle. It uses the **Union-Find (Disjoint Set)** data structure to detect cycles efficiently.

## Detailed Explanation

### Mechanism
1.  **Sort** all edges by weight (ascending).
2.  Initialize a **Union-Find** structure with each vertex in its own set.
3.  Iterate through sorted edges $(u, v)$:
    *   If `Find(u) != Find(v)` (they are in different sets):
        *   **Union(u, v)** (Connect them).
        *   Add edge to MST.
    *   Else (they are already connected):
        *   Skip edge (Adding it would form a cycle).

### Complexity
*   **Time**: $O(E \log E)$ or $O(E \log V)$ (dominated by sorting edges).
*   **Space**: $O(V + E)$.

## Code Examples (Go)
*Requires implementing Union-Find.*

```go
type Edge struct { u, v, w int }

func Kruskal(n int, edges []Edge) []Edge {
    sort.Slice(edges, func(i, j int) bool { return edges[i].w < edges[j].w })
    
    uf := NewUnionFind(n)
    var mst []Edge
    
    for _, e := range edges {
        if uf.Find(e.u) != uf.Find(e.v) {
            uf.Union(e.u, e.v)
            mst = append(mst, e)
        }
    }
    return mst
}
```

## Interview Questions

**Q: What is the main data structure used in Kruskal's?**
**A:** **Disjoint Set Union (DSU)** or Union-Find. It allows for near $O(1)$ cycle detection.

**Q: Can Kruskal's work on disconnected graphs?**
**A:** Yes, it will produce a **Minimum Spanning Forest** (a collection of MSTs for each connected component).

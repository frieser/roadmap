---
---

# Kruskal's Algorithm

## Abstract
**Kruskal's Algorithm** finds the **Minimum Spanning Tree (MST)** of a graph. It is a greedy algorithm that sorts all edges by weight and iteratively adds the smallest edge to the MST, provided it doesn't form a cycle. It is best suited for **sparse graphs**.

## Development

### Greedy Strategy
-   **Global Greedy**: Look at *all* edges in the entire graph.
-   **Choice**: Pick the absolute smallest edge available.
-   **Constraint**: If adding it creates a cycle (connects two nodes already in the same component), discard it.

### Implementation Details
-   **Sorting**: $O(E \log E)$.
-   **Cycle Detection**: Uses **Union-Find (Disjoint Set)** data structure ($O(\alpha(V)) \approx O(1)$).

## Code Examples (Go)

```go
type Edge struct {
    U, V, Weight int
}

func KruskalMST(n int, edges []Edge) []Edge {
    // 1. Sort edges by weight (Ascending)
    sort.Slice(edges, func(i, j int) bool {
        return edges[i].Weight < edges[j].Weight
    })

    mst := []Edge{}
    uf := NewUnionFind(n)

    // 2. Iterate and select
    for _, edge := range edges {
        if uf.Find(edge.U) != uf.Find(edge.V) {
            uf.Union(edge.U, edge.V)
            mst = append(mst, edge)
        }
    }
    return mst
}
```

## Go Application
-   **Network Design**: Laying cables to connect cities with min cost.
-   **Clustering**: Kruskal's can be stopped early ($k$ components left) to form $k$ clusters.

## Interview Preparation
1.  **Kruskal vs Prim**:
    -   Kruskal: Better for sparse graphs (edges sorted).
    -   Prim: Better for dense graphs (grows from one node).
2.  **Complexity**: Dominated by sorting edges: $O(E \log E)$.

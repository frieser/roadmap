---
---

# Prim's Algorithm

## Abstract
**Prim's Algorithm** is a greedy algorithm for finding the **Minimum Spanning Tree (MST)**. Unlike Kruskal's (which picks edges globally), Prim's "grows" the MST from a single starting vertex, always adding the cheapest edge connecting the tree to a non-tree vertex.

## Development

### Greedy Strategy
-   **Local Greedy**: Look at the "frontier" of the current tree.
-   **Choice**: Pick the node with smallest connection cost to the tree.
-   **Maintenance**: Use a Priority Queue to track min costs to all non-tree vertices.

### Complexity
-   **Time**: $O(E \log V)$ using Binary Heap. $O(E + V \log V)$ using Fibonacci Heap.
-   **Space**: $O(V)$.

## Code Examples (Go)

Structurally identical to Dijkstra, but `dist[v]` represents "min cost to connect $v$ to the MST", not "distance from source".

```go
func PrimMST(graph map[int]map[int]int) int {
    // Priority Queue stores (node, cost_to_connect)
    // visited set tracks MST inclusion
    // totalCost accumulates weights
    
    // Logic:
    // 1. Push arbitrary start node with cost 0
    // 2. While PQ not empty:
    //    Pop minimal cost node u
    //    If visited, skip
    //    Mark u visited, add cost to total
    //    For neighbors v of u:
    //       Push (v, weight) to PQ
    //       (Optimization: only if weight < current best cost to v)
    
    return totalCost
}
```

## Go Application
-   **Dense Graphs**: Prim's is generally preferred over Kruskal's for dense graphs ($E \approx V^2$) because it avoids sorting $E$ edges initially.

## Interview Preparation
1.  **Dijkstra vs Prim**:
    -   Dijkstra minimizes **path length from source** ($d[u] + w$).
    -   Prim minimizes **connection cost to tree** ($w$).
2.  **Cut Property**: Prim's relies on the cut property just like Kruskal's. The cut is between "Nodes in MST" and "Nodes outside".

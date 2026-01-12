---
---

# Dijkstra's Algorithm (Greedy Approach)

## Abstract
**Dijkstra's Algorithm** is the quintessential greedy algorithm for finding shortest paths in graphs with non-negative weights. While its graph-traversal logic is key, its classification as "Greedy" comes from the fact that at every step, it makes a **local optimal choice** (selecting the unvisited node with the smallest tentative distance) which leads to a **global optimal solution**.

## Development

### The Greedy Choice Property
At any point, we have a set of "visited" nodes (shortest path known) and "boundary" nodes (neighbors).
-   **Local Choice**: Pick the boundary node closest to the source.
-   **Why it works**: Since weights are non-negative, any path to this closest node through other unvisited nodes would necessarily be longer. Thus, the current shortest path is the *true* shortest path.

### Optimal Substructure
If the shortest path from $A$ to $C$ goes through $B$, then the sub-path from $A$ to $B$ is also the shortest path from $A$ to $B$. This property allows Dijkstra to build the solution incrementally.

### Comparison
| Algorithm | Approach | Constraint |
|-----------|----------|------------|
| **Dijkstra** | Greedy | Non-negative weights |
| **Bellman-Ford** | Dynamic Programming | No negative cycles |
| **A\*** | Heuristic Search | Admissible heuristic |

## Code Examples (Go)
*See `01-graphs/04-dijkstras-algorithm.md` for full implementation.*

Here we highlight the "Greedy Step" explicitly:

```go
// The Greedy Loop
for pq.Len() > 0 {
    // 1. GREEDY CHOICE: Pop the node with the SMALLEST distance
    u := heap.Pop(pq).(*Item)

    // 2. Optimization: If we found a better path to u already, skip
    if u.Priority > dist[u.Node] {
        continue
    }

    // 3. Relax neighbors (update tentative distances)
    for v, weight := range graph[u.Node] {
        if dist[u.Node]+weight < dist[v] {
            dist[v] = dist[u.Node] + weight
            heap.Push(pq, &Item{Node: v, Priority: dist[v]})
        }
    }
}
```

## Interview Preparation
1.  **Prove Greedy works**: Explain that with non-negative edges, extending a path can never reduce its length. Therefore, the "closest" node cannot be reached faster via a "farther" node.
2.  **Counter-example**: Draw a graph with a negative edge where Dijkstra fails (Greedy choice was wrong).

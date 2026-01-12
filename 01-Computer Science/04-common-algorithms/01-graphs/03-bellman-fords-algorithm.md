---
---

# Bellman-Ford Algorithm

## Abstract
The **Bellman-Ford Algorithm** is a graph search algorithm that finds the shortest path from a single source vertex to all other vertices in a weighted digraph. Unlike Dijkstra's algorithm, Bellman-Ford works correctly even with **negative edge weights**. It can also detect **negative cycles** in the graph.

## Development

### Core Concept
The algorithm relies on the principle of **relaxation**. It iteratively improves the distance estimates to all vertices.
1.  Initialize distances: Source = 0, others = $\infty$.
2.  Repeat $V-1$ times:
    -   For every edge $(u, v)$ with weight $w$:
        -   If $dist[u] + w < dist[v]$, update $dist[v] = dist[u] + w$.
3.  Check for negative cycles:
    -   Run one more relaxation step. If any distance decreases, a negative cycle exists.

### Complexity
- **Time**: $O(V \cdot E)$. Slower than Dijkstra ($O(E \log V)$).
- **Space**: $O(V)$.

## Code Examples (Go)

```go
package main

import (
    "fmt"
    "math"
)

type Edge struct {
    U, V, Weight int
}

func BellmanFord(edges []Edge, numVertices, start int) (map[int]float64, bool) {
    dist := make(map[int]float64)
    for i := 0; i < numVertices; i++ {
        dist[i] = math.Inf(1)
    }
    dist[start] = 0

    // Relax edges V-1 times
    for i := 0; i < numVertices-1; i++ {
        for _, edge := range edges {
            if dist[edge.U]+float64(edge.Weight) < dist[edge.V] {
                dist[edge.V] = dist[edge.U] + float64(edge.Weight)
            }
        }
    }

    // Check for negative cycles
    for _, edge := range edges {
        if dist[edge.U]+float64(edge.Weight) < dist[edge.V] {
            return nil, true // Negative cycle detected
        }
    }

    return dist, false
}
```

## Go Application
- **Routing Protocols**: Used in RIP (Routing Information Protocol).
- **Financial Arbitrage**: Detecting negative cycles corresponds to finding arbitrage opportunities in currency exchange graphs.

## Interview Preparation
1.  **Why V-1 iterations?**
    -   *Answer*: In a simple path (no cycles), the longest possible path has $V-1$ edges. Propagating the shortest path takes at most $V-1$ relaxations.
2.  **Dijkstra vs Bellman-Ford**:
    -   Dijkstra: Faster ($O(E \log V)$), no negative weights.
    -   Bellman-Ford: Slower ($O(VE)$), handles negative weights/cycles.

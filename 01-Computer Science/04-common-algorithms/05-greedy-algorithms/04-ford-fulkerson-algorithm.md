---
---

# Ford-Fulkerson Algorithm

## Abstract
The **Ford-Fulkerson Algorithm** computes the **Maximum Flow** in a flow network. It greedily finds "augmenting paths" from source to sink and pushes as much flow as possible along them until no more paths exist. The **Edmonds-Karp** implementation uses BFS to find the shortest augmenting path, ensuring termination.

## Development

### Core Concept
1.  Initialize flow = 0.
2.  **Residual Graph**: Tracks remaining capacity ($Capacity - Flow$).
3.  While there is a path from Source to Sink in Residual Graph:
    -   Find **bottleneck** capacity on this path.
    -   Augment flow by bottleneck amount.
    -   Update residual capacities (Forward edge -, Backward edge +).
4.  Return max flow.

### Greedy Logic
It assumes that pushing flow along *any* available path helps. The "backward edges" allow correcting "greedy mistakes" by effectively canceling previous flow.

### Complexity
-   **Ford-Fulkerson (DFS)**: $O(E \cdot f*)$ where $f*$ is max flow. Can differ badly.
-   **Edmonds-Karp (BFS)**: $O(V \cdot E^2)$. Strongly polynomial.

## Code Examples (Go)

```go
func MaxFlow(graph [][]int, s, t int) int {
    rGraph := copyGraph(graph) // Residual Graph
    parent := make([]int, len(graph))
    maxFlow := 0

    // BFS loop to find augmenting paths
    for bfs(rGraph, s, t, parent) {
        pathFlow := math.MaxInt32
        
        // Find bottleneck
        for v := t; v != s; v = parent[v] {
            u := parent[v]
            pathFlow = min(pathFlow, rGraph[u][v])
        }

        // Update residual capacities
        for v := t; v != s; v = parent[v] {
            u := parent[v]
            rGraph[u][v] -= pathFlow
            rGraph[v][u] += pathFlow
        }

        maxFlow += pathFlow
    }
    return maxFlow
}
```

## Go Application
-   **Traffic Routing**: Max throughput of cars.
-   **Bipartite Matching**: Maximum number of task assignments.
-   **Image Segmentation**: Min-Cut (Graph Cuts).

## Interview Preparation
1.  **Min-Cut Max-Flow Theorem**: The max amount of flow is equal to the capacity of the minimum cut (bottleneck) separating source and sink.
2.  **Infinite Loop**: Standard Ford-Fulkerson might not terminate with irrational capacities. Edmonds-Karp always terminates.

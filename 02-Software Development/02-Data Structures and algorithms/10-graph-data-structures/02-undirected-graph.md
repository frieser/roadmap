---
---

# Undirected Graph

## Summary
An **Undirected Graph** is a graph where edges have no direction. An edge $(u, v)$ is identical to $(v, u)$, implying a bidirectional relationship.

## Detailed Explanation

### Structure
*   **Edge $\{u, v\}$**: Connects $u$ and $v$. Can traverse $u \to v$ or $v \to u$.
*   **Degree**: The number of edges connected to a vertex (Total neighbors).
*   **Connected Graph**: A path exists between every pair of vertices.

### Use Cases
1.  **Social Networks**: Facebook (Friendship is mutual).
2.  **Physical Networks**: Cables connecting computers.
3.  **Road Maps**: Two-way streets.

## Code Examples (Go)

### Adjacency List
When adding an edge, we simply add it to **both** vertices' lists.

```go
package main

import "fmt"

type Graph struct {
    adjList map[int][]int
}

func NewGraph() *Graph {
    return &Graph{adjList: make(map[int][]int)}
}

func (g *Graph) AddEdge(u, v int) {
    // Add u -> v AND v -> u
    g.adjList[u] = append(g.adjList[u], v)
    g.adjList[v] = append(g.adjList[v], u)
}

func main() {
    g := NewGraph()
    g.AddEdge(1, 2) // 1 <-> 2
    
    fmt.Println(g.adjList)
}
```

## Interview Questions

**Q: What is the maximum number of edges in an undirected graph with $V$ vertices?**
**A:** $V(V-1) / 2$. This represents a **Complete Graph** ($K_V$).

**Q: How is cycle detection different in undirected graphs?**
**A:** Using DFS, if you encounter a visited neighbor that is **not** your immediate parent, a cycle exists. (You don't need the recursion stack tracking used in directed graphs).

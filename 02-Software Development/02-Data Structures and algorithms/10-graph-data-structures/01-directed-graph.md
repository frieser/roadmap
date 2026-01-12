---
---

# Directed Graph (Digraph)

## Summary
A **Directed Graph** (or Digraph) is a set of **vertices** (nodes) connected by **edges**, where each edge has a specific direction. An edge flows from a source vertex to a destination vertex.

## Detailed Explanation

### Structure
*   **Edge $(u, v)$**: Goes from $u \to v$.
    *   $u$ is the **Tail** (Source).
    *   $v$ is the **Head** (Destination).
*   **In-Degree**: Number of edges coming **into** a vertex.
*   **Out-Degree**: Number of edges going **out of** a vertex.
*   **Weighted**: Edges can carry values (weights), representing cost, distance, etc.

### Use Cases
1.  **Web Pages**: Link A $\to$ Link B (Hyperlinks).
2.  **Social Networks**: Twitter (Follower $\to$ Followee is one-way).
3.  **Dependencies**: Task A must finish before Task B.

## Code Examples (Go)

### Adjacency List
The most common representation. A map where keys are vertices and values are lists of destination vertices.

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
    // Only add u -> v (One way)
    g.adjList[u] = append(g.adjList[u], v)
}

func main() {
    g := NewGraph()
    g.AddEdge(1, 2) // 1 -> 2
    g.AddEdge(2, 3) // 2 -> 3
    g.AddEdge(3, 1) // 3 -> 1 (Cycle)
    
    fmt.Println(g.adjList)
}
```

## Interview Questions

**Q: What is the sum of all in-degrees in a directed graph?**
**A:** It is exactly equal to the number of edges $|E|$. Every edge has exactly one head.

**Q: How do you detect a cycle in a directed graph?**
**A:** Use **DFS**. Keep track of nodes in the current recursion stack (`onStack`). If you reach a node that is currently in the stack, a cycle exists.

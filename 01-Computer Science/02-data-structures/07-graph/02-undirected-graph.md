---
---

# Undirected Graph

## Abstract
An **Undirected Graph** is a graph where edges have **no direction**. The relationship between two vertices is symmetric: if Vertex A is connected to Vertex B, then Vertex B is connected to Vertex A. This structure is ideal for modeling mutual relationships, physical connections, or bidirectional networks.

## Development

### Core Concept
- **Edge**: An unordered pair $\{u, v\}$.
- **Degree**: The number of edges connected to a vertex. Since edges are bidirectional, we don't distinguish between "in" and "out".
- **Connectivity**: A graph is "connected" if there is a path between every pair of vertices.

### Properties
- **Handshaking Lemma**: The sum of degrees of all vertices is equal to $2 \times |E|$.
- **Max Edges**: A complete undirected graph with $V$ vertices has $V(V-1)/2$ edges.

### Real-world Analogies
- **Facebook**: Friendship is mutual. If A is friend with B, B is friend with A.
- **Physical Roads**: A two-way street network.
- **Network Cabling**: Physical cables connecting computers.

## Code Examples (Go)

### 1. Representation
We still use an Adjacency List, but when adding an edge, we add it to **both** vertices' lists.

```go
package main

import "fmt"

type Graph struct {
    adj map[int][]int
}

func NewGraph() *Graph {
    return &Graph{adj: make(map[int][]int)}
}

// AddEdge adds a bidirectional edge
func (g *Graph) AddEdge(u, v int) {
    g.adj[u] = append(g.adj[u], v)
    g.adj[v] = append(g.adj[v], u) // Symmetry
}

func main() {
    g := NewGraph()
    g.AddEdge(1, 2)
    
    fmt.Println(g.adj[1]) // [2]
    fmt.Println(g.adj[2]) // [1] - Automatically connected back
}
```

### 2. Connected Components (BFS)
Finding all connected sub-graphs within a disjoint graph.

```go
func FindComponents(g *Graph) [][]int {
    visited := make(map[int]bool)
    var components [][]int

    for node := range g.adj {
        if !visited[node] {
            // Start a new BFS/DFS for this component
            component := []int{}
            queue := []int{node}
            visited[node] = true

            for len(queue) > 0 {
                curr := queue[0]
                queue = queue[1:]
                component = append(component, curr)

                for _, neighbor := range g.adj[curr] {
                    if !visited[neighbor] {
                        visited[neighbor] = true
                        queue = append(queue, neighbor)
                    }
                }
            }
            components = append(components, component)
        }
    }
    return components
}
```

## Go Application
- **Peer-to-Peer Networks**: Modeling nodes in a P2P network (like BitTorrent clients) often uses undirected graphs for neighbor discovery.
- **Mesh Networks**: Service meshes or physical mesh topologies where links are bidirectional.

## Interview Preparation
1.  **Graph vs Tree**:
    -   *Answer*: A tree is a specific type of undirected graph that is connected and has **no cycles** (acyclic). It has exactly $V-1$ edges.
2.  **Detect Cycle in Undirected Graph**:
    -   *Logic*: Use DFS/Union-Find. During DFS, if you visit a node that is already visited and is **not** your immediate parent, a cycle exists.
3.  **Shortest Path**:
    -   *Unweighted*: BFS (Breadth-First Search) guarantees the shortest path in terms of number of edges.
    -   *Weighted*: Dijkstra's algorithm.

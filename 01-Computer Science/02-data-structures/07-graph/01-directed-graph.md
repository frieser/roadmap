---
---

# Directed Graph (Digraph)

## Abstract
A **Directed Graph** (or Digraph) is a set of vertices (nodes) connected by edges, where each edge has a **direction**. Unlike an undirected graph where an edge represents a two-way relationship, a directed edge from $A$ to $B$ ($A \rightarrow B$) allows travel only from $A$ to $B$, not vice versa. This structure models asymmetrical relationships like web page links, Twitter followers, or task dependencies.

## Development

### Core Concept
- **Vertex (V)**: A node in the graph.
- **Edge (E)**: An ordered pair $(u, v)$ representing a connection from start-node $u$ to end-node $v$.
- **Indegree**: Number of edges coming **into** a vertex.
- **Outdegree**: Number of edges going **out** of a vertex.

### Properties
- **Max Edges**: A directed graph with $V$ vertices can have at most $V(V-1)$ edges.
- **Cycles**: A path that starts and ends at the same vertex. A **Directed Acyclic Graph (DAG)** is a specific type of digraph with no cycles, crucial for scheduling (topological sort).

### Real-world Analogies
- **Twitter/Instagram**: User A follows User B ($A \rightarrow B$). This doesn't imply B follows A.
- **Web**: Page A links to Page B ($A \rightarrow B$).
- **Dependencies**: Package A imports Package B ($A \rightarrow B$).

## Code Examples (Go)

### 1. Representation (Adjacency List)
In Go, a directed graph is often represented as a map where keys are vertices and values are slices of destination vertices.

```go
package main

import "fmt"

// Graph represents a directed graph using an adjacency list
type Graph struct {
    adjacencyMap map[int][]int
}

// AddEdge adds a directed edge from u to v
func (g *Graph) AddEdge(u, v int) {
    if g.adjacencyMap == nil {
        g.adjacencyMap = make(map[int][]int)
    }
    g.adjacencyMap[u] = append(g.adjacencyMap[u], v)
}

func main() {
    g := &Graph{}
    g.AddEdge(1, 2) // 1 -> 2
    g.AddEdge(1, 3) // 1 -> 3
    g.AddEdge(2, 4) // 2 -> 4
    
    // Attempting to traverse back (2 -> 1) is impossible
    // unless explicit edge exists.
    fmt.Println("Neighbors of 1:", g.adjacencyMap[1]) // [2 3]
    fmt.Println("Neighbors of 2:", g.adjacencyMap[2]) // [4]
}
```

### 2. Indegree Calculation
Counting incoming edges is useful for finding "source" nodes (Indegree 0) or "sink" nodes (Outdegree 0).

```go
func CalculateIndegrees(g *Graph, numVertices int) map[int]int {
    indegree := make(map[int]int)
    for u, neighbors := range g.adjacencyMap {
        for _, v := range neighbors {
            indegree[v]++
        }
        // Ensure source nodes are initialized even if 0
        if _, exists := indegree[u]; !exists {
            indegree[u] = 0
        }
    }
    return indegree
}
```

## Go Application & Ecosystem
- **Go Modules**: The dependency graph of Go packages is a Directed Acyclic Graph (DAG). Go forbids cyclic imports to keep this property intact for faster compilation.
- **Concurrency**: Goroutines and Channels often form a directed data flow graph (Pipelines).
- **Terraform/Kubernetes**: Resource dependencies are modeled as DAGs to determine creation order.

## Interview Preparation
1.  **Detect Cycle in Directed Graph**:
    -   *Logic*: Use DFS with three states: `Unvisited`, `Visiting` (currently in recursion stack), `Visited`. If you see a `Visiting` node, cycle detected.
2.  **Topological Sort**:
    -   *Concept*: Linear ordering of vertices such that for every edge $u \rightarrow v$, $u$ comes before $v$. Only possible in DAGs.
    -   *Algorithm*: Kahn's Algorithm (using Indegrees) or DFS based.
3.  **Transposes**:
    -   *Question*: How to reverse a directed graph?
    -   *Answer*: Create a new graph. For every edge $u \rightarrow v$ in original, add $v \rightarrow u$ in new.

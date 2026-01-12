---
---

# Adjacency Matrix

## Abstract
An **Adjacency Matrix** is a 2D array of size $V \times V$ where $V$ is the number of vertices in the graph. The slot `matrix[i][j]` is **1** (or the edge weight) if there is an edge from vertex $i$ to vertex $j$, and **0** otherwise. It provides extremely fast lookups for edge existence but consumes quadratic memory.

## Development

### Structure
- **Data Structure**: `[][]int` or `[][]bool`.
- **Space Complexity**: $O(V^2)$. Always consumes this much space regardless of edge count.
- **Time Complexity**:
    -   **Add Edge**: $O(1)$.
    -   **Check Edge $(u, v)$**: $O(1)$.
    -   **Iterate Neighbors**: $O(V)$ (Must scan the entire row, even if empty).

### Pros vs Cons
| Pros | Cons |
|------|------|
| **O(1)** check to see if $A$ is connected to $B$. | **Wasted Space**: Consumes huge memory for sparse graphs. |
| Removing an edge is $O(1)$. | Adding a vertex is expensive ($O(V^2)$ copy). |
| Algebraic operations (Matrix Multiplication) can find paths. | Iterating neighbors is slow ($O(V)$ vs $O(degree)$). |

## Code Examples (Go)

### Basic Implementation

```go
package main

import "fmt"

type MatrixGraph struct {
    numVertices int
    matrix      [][]int // Using int allows storing weights. 0 = no edge.
}

func NewMatrixGraph(v int) *MatrixGraph {
    mat := make([][]int, v)
    for i := range mat {
        mat[i] = make([]int, v)
    }
    return &MatrixGraph{numVertices: v, matrix: mat}
}

func (g *MatrixGraph) AddEdge(u, v, weight int) {
    // Check bounds
    if u >= 0 && u < g.numVertices && v >= 0 && v < g.numVertices {
        g.matrix[u][v] = weight
        // g.matrix[v][u] = weight // Uncomment for Undirected
    }
}

func (g *MatrixGraph) HasEdge(u, v int) bool {
    return g.matrix[u][v] != 0
}

func main() {
    g := NewMatrixGraph(3)
    g.AddEdge(0, 2, 5) // Edge 0->2 with weight 5
    
    fmt.Println(g.HasEdge(0, 2)) // true
    fmt.Println(g.HasEdge(2, 0)) // false
}
```

## Go Application
- **Floyd-Warshall Algorithm**: The All-Pairs Shortest Path algorithm naturally uses an adjacency matrix (Dynamic Programming).
- **Dense Graphs**: In scenarios like "Complete Graphs" where every node connects to every other node, a matrix is more efficient than pointer chasing in lists.
- **Image Processing**: Pixels can be thought of as a grid (matrix) graph where each pixel connects to its neighbors.

## Interview Preparation
1.  **Matrix vs List**:
    -   *Question*: If you have a graph with 10,000 nodes and 20,000 edges, which representation should you use?
    -   *Answer*: Adjacency List. $V^2 = 100,000,000$ cells (approx 400MB-800MB RAM) for just 20,000 edges is extremely wasteful. List space is $\approx 20,000$, negligible.
2.  **Symmetry**:
    -   *Observation*: For an undirected graph, the Adjacency Matrix is symmetric along the main diagonal ($M[i][j] == M[j][i]$).

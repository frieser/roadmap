---
---

# Adjacency List

## Abstract
An **Adjacency List** is the most common way to represent a graph in memory. It represents a graph as an array (or map) of lists. The index of the array represents a vertex, and each element in its linked list (or slice) represents the other vertices that form an edge with the vertex. It is **space-efficient** for sparse graphs.

## Development

### Structure
- **Data Structure**: `map[int][]int` or `[][]int`.
- **Space Complexity**: $O(V + E)$. We only store edges that actually exist.
- **Time Complexity**:
    -   **Add Edge**: $O(1)$.
    -   **Check Edge $(u, v)$**: $O(degree(u))$ (Linear scan of $u$'s neighbors).
    -   **Iterate Neighbors**: $O(degree(u))$ (Very efficient).

### Pros vs Cons
| Pros | Cons |
|------|------|
| Saves memory for **Sparse Graphs** (few edges). | Checking if edge $(u, v)$ exists is slower than matrix ($O(1)$). |
| Faster to iterate over all edges. | Slightly more complex to implement generic delete. |
| Standard choice for most algorithms (BFS, DFS). | |

## Code Examples (Go)

### 1. Basic Slice of Slices (Dense Vertex IDs)
If vertices are numbered $0$ to $V-1$ sequentially.

```go
type Graph struct {
    V   int
    Adj [][]int
}

func NewGraph(V int) *Graph {
    adj := make([][]int, V)
    for i := range adj {
        adj[i] = make([]int, 0)
    }
    return &Graph{V: V, Adj: adj}
}

func (g *Graph) AddEdge(u, v int) {
    g.Adj[u] = append(g.Adj[u], v)
    // g.Adj[v] = append(g.Adj[v], u) // Uncomment for Undirected
}
```

### 2. Map-based (Sparse/Arbitrary IDs)
If vertex IDs are non-sequential (e.g., hash IDs, strings) or sparse.

```go
type SparseGraph struct {
    Adj map[string][]string
}

func (g *SparseGraph) AddEdge(u, v string) {
    if g.Adj == nil {
        g.Adj = make(map[string][]string)
    }
    g.Adj[u] = append(g.Adj[u], v)
}
```

## Go Application
- **Standard Library**: Go doesn't have a graph lib, but this map-based approach is idiomatic.
- **JSON Marshaling**: Adjacency lists map naturally to JSON objects (`{"user_1": ["user_2", "user_3"]}`), making them easy to serialize for APIs.

## Interview Preparation
1.  **When to use Adjacency List?**:
    -   *Answer*: Almost always, unless the graph is extremely dense (Edges $\approx V^2$) or you need $O(1)$ edge existence checks specifically.
2.  **Implementation**: Be comfortable writing the `map[int][]int` boilerplate from scratch.

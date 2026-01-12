---
---

# Spanning Tree & MST

## Abstract
A **Spanning Tree** of a connected, undirected graph is a subgraph that connects **all** the vertices together, without any cycles. It essentially "spans" the entire graph with the minimum number of edges ($V-1$). A **Minimum Spanning Tree (MST)** is a spanning tree where the sum of the weights of the edges is as small as possible. MSTs are fundamental in network design to minimize cost (cabling, piping, etc.).

## Development

### Core Concept
- **Vertices**: All $V$ vertices from the original graph must be present.
- **Edges**: Exactly $V-1$ edges.
- **Acyclic**: No loops allowed.
- **Weighted Graphs**: In graphs where edges have "costs" or "weights", finding the MST is an optimization problem.

### Algorithms
Two famous greedy algorithms solve the MST problem:

1.  **Kruskal's Algorithm**:
    -   Sort all edges by weight.
    -   Iterate through sorted edges and add them if they don't form a cycle.
    -   Uses **Union-Find** data structure for efficient cycle detection.
    
2.  **Prim's Algorithm**:
    -   Start from an arbitrary node.
    -   Grow the tree by always adding the cheapest edge connecting a tree node to a non-tree node.
    -   Uses a **Priority Queue** (Min-Heap).

## Code Examples (Go)

### Kruskal's Algorithm (Conceptual Implementation)
Requires a `UnionFind` structure (Disjoint Set).

```go
type Edge struct {
    U, V, Weight int
}

// UnionFind helper functions would go here (Find, Union)

func KruskalMST(vertices int, edges []Edge) []Edge {
    // 1. Sort edges by weight
    sort.Slice(edges, func(i, j int) bool {
        return edges[i].Weight < edges[j].Weight
    })

    mst := []Edge{}
    uf := NewUnionFind(vertices)

    for _, edge := range edges {
        // 2. If U and V are not connected, add edge
        if uf.Find(edge.U) != uf.Find(edge.V) {
            uf.Union(edge.U, edge.V)
            mst = append(mst, edge)
        }
    }
    return mst
}
```

### Prim's Algorithm Idea
Using Go's `container/heap`:

1.  Push all edges from start node to Min-Heap.
2.  Pop cheapest edge $(u, v)$.
3.  If $v$ is visited, ignore.
4.  Mark $v$ visited, add to MST cost, push all edges from $v$ to Heap.

## Go Application
- **Network Routing**: Spanning Tree Protocol (STP) in Ethernet switches prevents bridge loops.
- **Cluster Management**: Determining the minimal connection required to keep a distributed system connected.
- **Maze Generation**: Randomized Prim's or Kruskal's are popular algorithms for generating perfect mazes (mazes with one solution and no loops).

## Interview Preparation
1.  **MST Uniqueness**:
    -   *Question*: Is the MST unique?
    -   *Answer*: If all edge weights are distinct, yes. If some weights are equal, there can be multiple valid MSTs.
2.  **Cut Property**:
    -   *Concept*: For any cut (a split of vertices into two disjoint sets), the minimum weight edge crossing the cut **must** belong to the MST.
3.  **Maximum Spanning Tree**:
    -   *Logic*: Same algorithms, but sort edges descending (Kruskal) or use Max-Heap (Prim).

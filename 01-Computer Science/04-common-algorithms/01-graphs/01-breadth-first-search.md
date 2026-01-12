---
---

# Breadth First Search (BFS)

## Abstract
**Breadth First Search (BFS)** is a fundamental graph traversal algorithm that explores vertices in the order of their distance from the source vertex. It visits all neighbors of a vertex before moving to their neighbors. It is typically used to find the **shortest path** in unweighted graphs.

## Development

### Core Concept
BFS uses a **Queue** (First-In-First-Out) data structure to keep track of the next vertices to visit.
1.  Start at the source node.
2.  Enqueue the source and mark it as visited.
3.  While the queue is not empty:
    -   Dequeue a vertex $u$.
    -   Process $u$.
    -   For every neighbor $v$ of $u$:
        -   If $v$ is not visited, mark it as visited and enqueue it.

### Complexity
- **Time**: $O(V + E)$ where $V$ is vertices and $E$ is edges.
- **Space**: $O(V)$ to store the queue and visited set.

## Code Examples (Go)

### BFS Implementation
```go
package main

import "fmt"

func BFS(graph map[int][]int, start int) {
    visited := make(map[int]bool)
    queue := []int{start}
    visited[start] = true

    for len(queue) > 0 {
        // Dequeue
        curr := queue[0]
        queue = queue[1:]

        fmt.Printf("Visited: %d\n", curr)

        for _, neighbor := range graph[curr] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
}

func main() {
    graph := map[int][]int{
        0: {1, 2},
        1: {2},
        2: {0, 3},
        3: {3},
    }
    BFS(graph, 2)
}
```

## Go Application & Ecosystem
- **Web Crawlers**: Discovering pages level by level.
- **Garbage Collection**: Identifying reachable objects (though Go uses a variation of Tricolor Mark-and-Sweep which is conceptually a traversal).
- **Networking**: Broadcasting packets (flooding).

## Interview Preparation
1.  **BFS vs DFS?**
    -   *Answer*: BFS finds the shortest path in unweighted graphs and explores neighbors first. DFS goes deep and is better for pathfinding in mazes or topological sorting.
2.  **Memory Usage**: BFS can consume more memory ($O(Width)$) than DFS ($O(Depth)$) if the graph is very wide.
3.  **Level Order Traversal**: BFS on a tree is exactly Level Order Traversal.

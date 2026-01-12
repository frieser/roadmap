---
---

# Depth First Search (DFS)

## Abstract
**Depth First Search (DFS)** is a graph traversal algorithm that explores as far as possible along each branch before backtracking. It dives deep into the graph structure immediately. It is useful for topological sorting, detecting cycles, and solving maze-like puzzles.

## Development

### Core Concept
DFS uses a **Stack** (Last-In-First-Out) data structure, either explicitly or implicitly via the **Call Stack** (recursion).
1.  Start at source node.
2.  Mark as visited.
3.  For every neighbor $v$:
    -   If $v$ is not visited, recursively call DFS on $v$.

### Complexity
- **Time**: $O(V + E)$.
- **Space**: $O(V)$ in worst case (recursion stack).

## Code Examples (Go)

### Recursive DFS
```go
package main

import "fmt"

func DFS(graph map[int][]int, curr int, visited map[int]bool) {
    if visited[curr] {
        return
    }
    visited[curr] = true
    fmt.Printf("Visited: %d\n", curr)

    for _, neighbor := range graph[curr] {
        if !visited[neighbor] {
            DFS(graph, neighbor, visited)
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
    visited := make(map[int]bool)
    DFS(graph, 2, visited)
}
```

### Iterative DFS (Using Stack)
```go
func DFSIterative(graph map[int][]int, start int) {
    visited := make(map[int]bool)
    stack := []int{start}

    for len(stack) > 0 {
        // Pop
        curr := stack[len(stack)-1]
        stack = stack[:len(stack)-1]

        if !visited[curr] {
            visited[curr] = true
            fmt.Printf("Visited: %d\n", curr)

            // Push neighbors (reverse order to maintain order)
            for _, neighbor := range graph[curr] {
                if !visited[neighbor] {
                    stack = append(stack, neighbor)
                }
            }
        }
    }
}
```

## Go Application
- **Cycle Detection**: Used in `go mod graph` dependency analysis.
- **Topological Sort**: Build systems (determining compilation order).
- **Maze Solving**: Backtracking algorithms are essentially DFS.

## Interview Preparation
1.  **Can DFS find shortest path?**
    -   *Answer*: No, it finds *a* path, but not necessarily the shortest.
2.  **Stack Overflow**: Deep recursion on large graphs can crash the program. Go's stack is dynamic and can grow large (1GB), but it's not infinite.
3.  **Detecting Cycles**: In a directed graph, if you hit a node that is currently in the recursion stack (visiting state), a cycle exists.

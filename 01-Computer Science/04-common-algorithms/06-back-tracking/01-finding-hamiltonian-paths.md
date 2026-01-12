---
---

# Finding Hamiltonian Paths

## Abstract
A **Hamiltonian Path** is a path in a directed or undirected graph that visits each vertex **exactly once**. A **Hamiltonian Cycle** is a Hamiltonian Path that is a cycle (returns to start). Finding such a path is an **NP-Complete** problem, typically solved via **Backtracking**.

## Development

### Backtracking Logic
1.  **Choice**: Pick a neighbor of the current vertex.
2.  **Constraint**: Neighbor must not have been visited yet.
3.  **Goal**: If path length == $V$, we found a path.
4.  **Backtrack**: If recursion fails, unmark neighbor as visited and try next neighbor.

### Complexity
-   **Time**: $O(N!)$. We try permutations of vertices.
-   **Space**: $O(N)$ (Recursion stack).

## Code Examples (Go)

```go
package main

func FindHamiltonianPath(graph [][]int) []int {
    n := len(graph)
    path := make([]int, 0, n)
    visited := make([]bool, n)

    var backtrack func(u int) bool
    backtrack = func(u int) bool {
        path = append(path, u)
        visited[u] = true

        if len(path) == n {
            return true // Found
        }

        for _, v := range graph[u] {
            if !visited[v] {
                if backtrack(v) {
                    return true
                }
            }
        }

        // Backtrack
        visited[u] = false
        path = path[:len(path)-1]
        return false
    }

    // Try starting from every node
    for i := 0; i < n; i++ {
        if backtrack(i) {
            return path
        }
    }
    return nil
}
```

## Go Application
-   **Traveling Salesman Problem (TSP)**: Finding the *shortest* Hamiltonian cycle.
-   **Logistics**: Route planning where every stop must be visited once.

## Interview Preparation
1.  **Eulerian vs Hamiltonian**:
    -   Eulerian: Visit every **Edge** once ($P$ problem, fast).
    -   Hamiltonian: Visit every **Vertex** once ($NP$ problem, slow).
2.  **Optimization**: Dynamic Programming (Held-Karp) reduces complexity to $O(N^2 2^N)$.

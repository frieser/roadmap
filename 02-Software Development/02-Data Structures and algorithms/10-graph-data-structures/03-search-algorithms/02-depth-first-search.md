---
---

# Depth First Search (DFS) - Graph

## Summary
**DFS** explores as deep as possible along each branch before backtracking. It is useful for topological sorting, cycle detection, and connectivity checks.

## Detailed Explanation

### Mechanism
1.  **Stack/Recursion**: Stores the path history.
2.  **Visited Set**: Essential to prevent infinite loops in cyclic graphs.
3.  **Process**: Visit $u$, recursively visit unvisited neighbor $v$.

### Complexity
*   **Time**: $O(V + E)$.
*   **Space**: $O(V)$ (Recursion depth).

## Code Examples (Go)

```go
func DFS(graph map[int][]int, start int, visited map[int]bool) {
    visited[start] = true
    fmt.Printf("Visited: %d\n", start)
    
    for _, neighbor := range graph[start] {
        if !visited[neighbor] {
            DFS(graph, neighbor, visited)
        }
    }
}
```

## Interview Questions

**Q: When is DFS preferred over BFS?**
**A:**
1.  **Memory**: If the graph is very wide but not deep, DFS uses less memory ($O(depth)$ vs $O(width)$).
2.  **Topological Sort**: DFS naturally produces a topological ordering (reverse post-order).
3.  **Maze Solving**: DFS is great for finding *any* path (not necessarily the shortest).

**Q: How do you perform a Topological Sort using DFS?**
**A:** Run DFS. When returning from the recursive call (finishing a node), push that node onto a stack. The stack will contain the topological sort order.

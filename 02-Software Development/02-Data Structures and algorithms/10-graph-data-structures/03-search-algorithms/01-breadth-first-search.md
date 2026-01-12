---
---

# Breadth First Search (BFS) - Graph

## Summary
**BFS** explores a graph layer by layer, starting from a source node. In unweighted graphs, it guarantees finding the **shortest path** (fewest edges) to any reachable node.

## Detailed Explanation

### Mechanism
1.  **Queue**: Stores nodes to visit next (FIFO).
2.  **Visited Set**: Tracks visited nodes to prevent cycles and redundant work.
3.  **Process**: Dequeue node $u$, visit all unvisited neighbors $v$, mark $v$ visited, enqueue $v$.

### Complexity
*   **Time**: $O(V + E)$ (Visit every vertex and edge once).
*   **Space**: $O(V)$ (Queue size).

## Code Examples (Go)

```go
func BFS(graph map[int][]int, start int) {
    visited := make(map[int]bool)
    queue := []int{start}
    visited[start] = true
    
    for len(queue) > 0 {
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
```

## Interview Questions

**Q: Can BFS find the shortest path in a weighted graph?**
**A:** No. BFS only counts "hops" (edges). It assumes all edges cost 1. For weighted graphs, use **Dijkstra's Algorithm**.

**Q: What happens if the graph is disconnected?**
**A:** Standard BFS only visits nodes reachable from the `start` node. To visit the whole graph, loop through all nodes and launch BFS if unvisited.

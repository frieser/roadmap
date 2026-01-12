---
---

# Dijkstra's Algorithm

## Summary
**Dijkstra's Algorithm** finds the shortest paths from a source node to all other nodes in a graph with **non-negative** edge weights. It is a "Greedy" algorithm that always visits the unvisited node with the smallest known distance.

## Detailed Explanation

### Mechanism
1.  **Priority Queue (Min-Heap)**: Stores nodes ordered by current shortest distance.
2.  **Distances**: Initialize source to 0, all others to $\infty$.
3.  While Queue not empty:
    *   Pop node $u$ with smallest distance.
    *   For each neighbor $v$ of $u$:
        *   `newDist = dist[u] + weight(u, v)`
        *   If `newDist < dist[v]`: Update `dist[v]` and push $v$ to Queue.

### Complexity
*   **Time**: $O(E \log V)$ using a Binary Heap.
*   **Space**: $O(V + E)$.

## Code Examples (Go)
*Requires implementing `heap.Interface`.*

```go
// Conceptual snippet
func Dijkstra(graph Graph, start Node) map[Node]int {
    dist := make(map[Node]int) // Init to infinity
    pq := &MinHeap{} 
    
    heap.Push(pq, Item{node: start, dist: 0})
    
    for pq.Len() > 0 {
        u := heap.Pop(pq).(Item)
        
        if u.dist > dist[u.node] { continue } // Stale entry
        
        for v, weight := range graph[u.node] {
            if dist[u.node] + weight < dist[v] {
                dist[v] = dist[u.node] + weight
                heap.Push(pq, Item{node: v, dist: dist[v]})
            }
        }
    }
    return dist
}
```

## Interview Questions

**Q: Why doesn't Dijkstra work with negative weights?**
**A:** Dijkstra assumes that once a node is "visited" (popped from PQ), its optimal distance is finalized. Negative edges can break this assumption by providing a "shortcut" back to an already finalized node, which Dijkstra won't re-evaluate correctly. Use **Bellman-Ford** instead.

**Q: What is the difference between Dijkstra and Prim's Algorithm?**
**A:**
*   **Dijkstra**: Finds Shortest Path (Source $\to$ Target).
*   **Prim**: Finds Minimum Spanning Tree (Connects all nodes with min total weight).
*   Logic is very similar, but Dijkstra adds `total_distance_so_far + edge`, while Prim only cares about `edge_weight`.

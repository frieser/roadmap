---
---

# A* Search Algorithm

## Abstract
**A* (A-Star)** is an informed search algorithm used for pathfinding and graph traversal. It is an extension of Dijkstra's algorithm that achieves better performance by using a **heuristic function** to estimate the cost from the current node to the goal. It finds the shortest path while exploring fewer nodes than Dijkstra.

## Development

### Core Concept
A* selects the path that minimizes:
$$ f(n) = g(n) + h(n) $$
-   $g(n)$: Exact cost from start to node $n$.
-   $h(n)$: Heuristic estimated cost from $n$ to goal.

The heuristic $h(n)$ must be **admissible** (never overestimate the true cost) to guarantee the shortest path.

### Complexity
- **Time**: Depends on the heuristic. In worst case (heuristic = 0), it becomes Dijkstra ($O(E \log V)$).
- **Space**: $O(V)$ (stores all generated nodes in open set).

## Code Examples (Go)

Structurally similar to Dijkstra, but priority is $f(n)$.

```go
// Heuristic (Manhattan distance for grid)
func heuristic(a, b Point) float64 {
    return math.Abs(float64(a.X - b.X)) + math.Abs(float64(a.Y - b.Y))
}

// Pseudo-code for A* loop
func AStar(start, goal Point) {
    openSet := &PriorityQueue{} // Priority = f_score
    heap.Push(openSet, &Item{Val: start, Priority: 0})
    
    gScore := make(map[Point]float64)
    gScore[start] = 0

    for openSet.Len() > 0 {
        current := heap.Pop(openSet).(*Item).Val

        if current == goal {
            return reconstructPath(cameFrom, current)
        }

        for _, neighbor := range getNeighbors(current) {
            tentativeG := gScore[current] + dist(current, neighbor)
            
            if tentativeG < gScore[neighbor] {
                cameFrom[neighbor] = current
                gScore[neighbor] = tentativeG
                fScore := tentativeG + heuristic(neighbor, goal)
                
                // Add or update in priority queue
                heap.Push(openSet, &Item{Val: neighbor, Priority: fScore})
            }
        }
    }
}
```

## Go Application
- **Game Development**: Pathfinding for NPCs in grid-based games.
- **Robotics**: Navigation and motion planning.
- **Maps**: Calculating routes with estimated travel times.

## Interview Preparation
1.  **What happens if h(n) = 0?**
    -   *Answer*: A* behaves exactly like Dijkstra's algorithm.
2.  **What happens if h(n) is very high?**
    -   *Answer*: It becomes a Greedy Best-First Search, which is fast but not guaranteed to find the shortest path.
3.  **Admissibility**: Why is it important? If $h(n)$ overestimates, A* might skip the actual shortest path thinking it's too expensive.

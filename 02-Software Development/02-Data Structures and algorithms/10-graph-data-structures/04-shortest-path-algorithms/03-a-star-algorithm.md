---
---

# A* Search Algorithm

## Summary
**A* (A-Star)** is an informed search algorithm used for pathfinding. It improves upon Dijkstra by using a **Heuristic** to estimate the cost from the current node to the goal, prioritizing paths that seem more promising.

## Detailed Explanation

### The Formula
$$ f(n) = g(n) + h(n) $$
*   $g(n)$: Actual cost from start to node $n$.
*   $h(n)$: **Heuristic** estimated cost from $n$ to goal.
*   $f(n)$: Total estimated cost.

### Heuristics
*   **Euclidean Distance**: Straight line (for flight paths).
*   **Manhattan Distance**: Grid movement (for city blocks).

### Complexity
*   **Time**: Depends heavily on the heuristic. In worst case (bad heuristic), it degrades to Dijkstra or BFS.
*   **Optimality**: A* is optimal if the heuristic is **admissible** (never overestimates the cost).

## Use Cases
1.  **Video Games**: Unit movement (Pathfinding).
2.  **Robotics**: Navigation.
3.  **Maps**: GPS Routing.

## Interview Questions

**Q: What happens if $h(n) = 0$?**
**A:** A* becomes **Dijkstra’s Algorithm**. It guarantees the shortest path but explores more nodes than necessary.

**Q: What is an "Admissible" heuristic?**
**A:** A heuristic that *never* overestimates the true cost to reach the goal. If $h(n)$ says "10 miles" but the real distance is "12 miles", it's admissible. If it says "15 miles", it's not admissible, and A* might fail to find the optimal path.

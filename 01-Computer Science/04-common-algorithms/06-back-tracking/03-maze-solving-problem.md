---
---

# Maze Solving Problem (Rat in a Maze)

## Abstract
**Maze Solving** involves finding a path from a starting point to a destination in a grid (maze) blocked by walls. The "Rat in a Maze" problem typically allows movement in 2 (Down, Right) or 4 (Up, Down, Left, Right) directions. It is solved using backtracking to explore paths and retreating from dead ends.

## Development

### Backtracking Logic
1.  **Choice**: Move to neighbor $(r, c)$.
2.  **Constraint**:
    -   Is $(r, c)$ within bounds?
    -   Is $(r, c)$ a wall?
    -   Has $(r, c)$ already been visited in current path?
3.  **Goal**: If $(r, c) == Destination$, return true.
4.  **Backtrack**: Mark $(r, c)$ as unvisited to allow other paths to use it (or keep visited if finding *any* path is sufficient).

### Complexity
-   **Time**: $O(2^{N^2})$ or $O(4^{N^2})$ in worst case (exponential).
-   **Space**: $O(N^2)$ path length.

## Code Examples (Go)

```go
func SolveMaze(maze [][]int) bool {
    N := len(maze)
    path := make([][]int, N)
    for i := range path { path[i] = make([]int, N) }

    var solve func(x, y int) bool
    solve = func(x, y int) bool {
        // Goal Reached
        if x == N-1 && y == N-1 {
            path[x][y] = 1
            return true
        }

        // Check Bounds & Wall
        if x >= 0 && x < N && y >= 0 && y < N && maze[x][y] == 1 {
            path[x][y] = 1 // Mark path

            // Move Right
            if solve(x, y+1) { return true }
            // Move Down
            if solve(x+1, y) { return true }

            path[x][y] = 0 // Backtrack
            return false
        }
        return false
    }

    return solve(0, 0)
}
```

## Go Application
-   **Robotics**: Path planning.
-   **Image Analysis**: Pixel connectivity.

## Interview Preparation
1.  **BFS vs DFS**:
    -   BFS guarantees shortest path.
    -   DFS (Backtracking) does not, but uses less memory and is simpler to implement recursively.
2.  **Dead Ends**: Backtracking efficiently prunes dead ends.

---
---

# The Knight's Tour Problem

## Abstract
The **Knight's Tour** is a classic chess puzzle: move a knight to every square on a chessboard exactly once. If the knight ends on a square attacking the starting square, it's a **closed** tour (cycle); otherwise, it's **open**. It is a specific instance of the Hamiltonian Path problem.

## Development

### Warnsdorff's Rule (Heuristic)
Pure backtracking is too slow for large boards ($8 \times 8$).
**Heuristic**: Always move to the square with the **fewest onward moves**. This greedy heuristic allows finding tours in linear time $O(N)$ for most starts.

### Backtracking Logic
1.  **Choice**: Move to valid L-shape square.
2.  **Constraint**: Square not visited.
3.  **Goal**: `move_count == 64`.
4.  **Backtrack**: Unmark square.

## Code Examples (Go)

```go
var moves = [][2]int{
    {2, 1}, {1, 2}, {-1, 2}, {-2, 1},
    {-2, -1}, {-1, -2}, {1, -2}, {2, -1},
}

func SolveKnightsTour(boardSize int) {
    grid := make([][]int, boardSize)
    for i := range grid { grid[i] = make([]int, boardSize) }

    var solve func(x, y, count int) bool
    solve = func(x, y, count int) bool {
        grid[x][y] = count
        
        if count == boardSize*boardSize {
            return true
        }

        // Try all 8 moves
        for _, m := range moves {
            nx, ny := x+m[0], y+m[1]
            if isValid(nx, ny, boardSize, grid) {
                if solve(nx, ny, count+1) {
                    return true
                }
            }
        }

        grid[x][y] = 0 // Backtrack
        return false
    }

    solve(0, 0, 1)
}
```

## Go Application
-   **Puzzle Games**: Generating levels.
-   **Graph Theory**: Testing Hamiltonian path algorithms.

## Interview Preparation
1.  **Why is it hard?**: The branching factor is up to 8, and depth is 64. $8^{64}$ is huge.
2.  **Warnsdorff's Rule**: Essential to mention for efficient solution. Without it, backtracking usually fails (times out) on $8 \times 8$.

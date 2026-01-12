---
---

# Island Traversal (Grid Search)

## Summary
**Island Traversal** (or Grid DFS/BFS) is a technique used to solve problems on a 2D grid (matrix), such as "Number of Islands", "Max Area of Island", or "Shortest Path in Maze". It treats the grid as a graph where each cell is a node and neighbors (Up, Down, Left, Right) are edges.

## Detailed Explanation

### Mechanism
Iterate through every cell in the grid. If a cell meets a condition (e.g., is '1' or 'Land'):
1.  Start a traversal (DFS or BFS) from that cell.
2.  Mark the cell and all connected valid neighbors as **Visited** (e.g., turn '1' to '0').
3.  Increment island count.

## Code Examples (Go)

### Number of Islands (DFS)
```go
func NumIslands(grid [][]byte) int {
    if len(grid) == 0 { return 0 }
    count := 0
    
    for r := 0; r < len(grid); r++ {
        for c := 0; c < len(grid[0]); c++ {
            if grid[r][c] == '1' {
                count++
                dfs(grid, r, c)
            }
        }
    }
    return count
}

func dfs(grid [][]byte, r, c int) {
    // Bounds check and visited check
    if r < 0 || c < 0 || r >= len(grid) || c >= len(grid[0]) || grid[r][c] == '0' {
        return
    }
    
    grid[r][c] = '0' // Mark visited (sink the island)
    
    // Visit neighbors
    dfs(grid, r+1, c)
    dfs(grid, r-1, c)
    dfs(grid, r, c+1)
    dfs(grid, r, c-1)
}
```

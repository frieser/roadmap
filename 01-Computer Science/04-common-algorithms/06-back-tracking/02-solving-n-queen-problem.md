---
---

# Solving N-Queen Problem

## Abstract
The **N-Queen Problem** is the challenge of placing $N$ chess queens on an $N \times N$ chessboard so that no two queens threaten each other. Thus, a solution requires that no two queens share the same row, column, or diagonal. It is a classic backtracking problem.

## Development

### Backtracking Template
1.  **Choice**: Place queen in column $j$ of current row $i$.
2.  **Constraint**: Check `isValid(row, col)`.
    -   No queen in column $j$ above.
    -   No queen in top-left diagonal.
    -   No queen in top-right diagonal.
3.  **Goal**: If `row == N`, solution found.
4.  **Backtrack**: Remove queen, try next column.

### Complexity
-   **Time**: $O(N!)$.
-   **Space**: $O(N)$ for board/recursion.

## Code Examples (Go)

```go
package main

import "fmt"

func SolveNQueens(n int) [][]string {
    board := make([][]rune, n)
    for i := range board {
        board[i] = make([]rune, n)
        for j := range board[i] {
            board[i][j] = '.'
        }
    }
    
    var solutions [][]string
    
    // Helper to check validity
    isValid := func(row, col int) bool {
        // Check Col
        for i := 0; i < row; i++ {
            if board[i][col] == 'Q' { return false }
        }
        // Check Diagonals
        for i, j := row-1, col-1; i >= 0 && j >= 0; i, j = i-1, j-1 {
            if board[i][j] == 'Q' { return false }
        }
        for i, j := row-1, col+1; i >= 0 && j < n; i, j = i-1, j+1 {
            if board[i][j] == 'Q' { return false }
        }
        return true
    }

    var backtrack func(row int)
    backtrack = func(row int) {
        if row == n {
            // Found solution, snapshot board
            return
        }
        for col := 0; col < n; col++ {
            if isValid(row, col) {
                board[row][col] = 'Q'
                backtrack(row + 1)
                board[row][col] = '.' // Backtrack
            }
        }
    }
    
    backtrack(0)
    return solutions
}
```

## Go Application
-   **Constraint Satisfaction**: General template for CSPs.
-   **Benchmarks**: Often used to benchmark recursion speed.

## Interview Preparation
1.  **Optimization**: Use boolean arrays (bitmasks) for columns and diagonals (`cols`, `d1`, `d2`) to make validity check $O(1)$ instead of $O(N)$.
2.  **Counting Solutions**: For $N=8$, there are 92 solutions.

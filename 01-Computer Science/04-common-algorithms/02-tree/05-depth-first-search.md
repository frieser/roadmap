---
---

# Depth First Search (DFS)

## Abstract
**Depth First Search (DFS)** for trees is a category of traversals that includes **Pre-Order**, **In-Order**, and **Post-Order**. It explores a branch as deep as possible before backtracking. Since trees are acyclic graphs, DFS on a tree is simply a specific ordering of visiting nodes without the need for cycle detection.

## Development

### Core Concept
DFS prioritizes depth. It uses a **Stack** (implicit or explicit).
- **Pre-Order**: Useful for copying.
- **In-Order**: Useful for sorting (BST).
- **Post-Order**: Useful for deletion/evaluation.

### DFS vs BFS for Trees
| Feature | DFS | BFS |
|---------|-----|-----|
| **Data Structure** | Stack (Recursion) | Queue |
| **Space** | $O(h)$ (Height) | $O(w)$ (Width) |
| **Best For** | Path finding, Searching deep | Shortest path, Level processing |
| **Recursion** | Natural | Unnatural |

## Code Examples (Go)
*See Pre/In/Post-Order files for specific implementations.*

Here is a generic DFS to find a path to a target value:

```go
func FindPath(root *TreeNode, target int) []int {
    var path []int
    var dfs func(node *TreeNode) bool
    
    dfs = func(node *TreeNode) bool {
        if node == nil {
            return false
        }
        
        path = append(path, node.Val)
        
        if node.Val == target {
            return true
        }
        
        if dfs(node.Left) || dfs(node.Right) {
            return true
        }
        
        // Backtrack
        path = path[:len(path)-1]
        return false
    }
    
    if dfs(root) {
        return path
    }
    return nil
}
```

## Go Application
- **Backtracking Algorithms**: Solving puzzles (N-Queens, Sudoku) uses DFS logic on a state-space tree.
- **Recursion Limits**: Be aware of `debug.SetMaxStack` if processing extremely deep trees, though Go's stack is resizable and generous.

## Interview Preparation
1.  **Maximum Depth**: `return 1 + max(depth(left), depth(right))` is a classic DFS application.
2.  **Balanced Binary Tree**: Check if height difference between left and right subtrees is $\le 1$ for every node (DFS Post-Order).

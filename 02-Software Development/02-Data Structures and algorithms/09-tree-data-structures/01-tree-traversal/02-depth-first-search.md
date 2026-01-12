---
---

# Depth First Search (DFS)

## Summary
**Depth First Search (DFS)** is a traversal strategy that explores as far as possible along each branch before backtracking. It dives "deep" into the tree first.

## Detailed Explanation

### Mechanism
DFS uses a **Stack** (LIFO) or **Recursion** (Call Stack).
There are three common variations for Binary Trees:
1.  **Pre-Order**: Root, Left, Right.
2.  **In-Order**: Left, Root, Right.
3.  **Post-Order**: Left, Right, Root.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time** | $O(n)$ | Visits every node once. |
| **Space** | $O(h)$ | Where $h$ is the height (Stack depth). |

## Use Cases
1.  **Path Finding**: Solving mazes (backtracking).
2.  **Topological Sort**: Scheduling dependencies.
3.  **Tree Analysis**: Checking height, validating BST properties.

## Code Examples (Go)
*See specific files for Pre/In/Post order implementations.*

### Generic Recursive Structure
```go
func DFS(node *Node) {
    if node == nil {
        return
    }
    // Pre-Order Logic Here
    DFS(node.Left)
    // In-Order Logic Here
    DFS(node.Right)
    // Post-Order Logic Here
}
```

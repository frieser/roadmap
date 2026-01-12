---
---

# Binary Trees

## Summary
A **Binary Tree** is a hierarchical data structure in which each node has at most two children, referred to as the **left child** and the **right child**. It is the foundational structure for more advanced trees like BSTs, AVL Trees, and Heaps.

## Detailed Explanation

### Structure
*   **Root**: The topmost node.
*   **Leaf**: A node with no children.
*   **Height**: The number of edges from the root to the deepest leaf.
*   **Depth**: The number of edges from the root to a specific node.

### Types of Binary Trees
1.  **Full**: Every node has either 0 or 2 children.
2.  **Complete**: All levels are completely filled except possibly the last, which is filled from left to right. (Crucial for Heaps).
3.  **Perfect**: All internal nodes have 2 children and all leaves are at the same level.
4.  **Balanced**: The height of the left and right subtrees of any node differ by at most 1.

### Complexity
| Operation | Time | Space | Notes |
| :--- | :--- | :--- | :--- |
| **Search** | $O(n)$ | $O(h)$ | Must traverse potentially all nodes. |
| **Insert** | $O(n)$ | $O(h)$ | Finding insertion point takes time. |
| **Delete** | $O(n)$ | $O(h)$ | |

*Note: Without specific ordering properties (like in a BST), a Binary Tree is just a structural container.*

## Code Examples (Go)

### Node Definition
Go uses `struct` pointers to link nodes.

```go
package main

import "fmt"

type Node struct {
    Value int
    Left  *Node
    Right *Node
}

func main() {
    // Constructing a simple tree
    //       1
    //      / \
    //     2   3
    root := &Node{Value: 1}
    root.Left = &Node{Value: 2}
    root.Right = &Node{Value: 3}
    
    fmt.Printf("Root: %d, Left: %d, Right: %d\n", root.Value, root.Left.Value, root.Right.Value)
}
```

## Interview Questions

**Q: What is the maximum number of nodes in a binary tree of height $h$?**
**A:** $2^{h+1} - 1$. For example, height 0 (1 node) -> $2^1 - 1 = 1$. Height 2 (3 levels) -> $2^3 - 1 = 7$.

**Q: What is the difference between a Complete and a Full Binary Tree?**
**A:** A **Full** tree has no nodes with only 1 child. A **Complete** tree fills levels strictly from top-to-bottom, left-to-right (used in Array-based Heaps).

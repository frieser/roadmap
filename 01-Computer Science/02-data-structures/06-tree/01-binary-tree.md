---
---

# Binary Tree

## Abstract
A **Binary Tree** is a hierarchical data structure in which each node has at most two children, referred to as the **left child** and the **right child**. It is the fundamental building block for more complex tree-based structures like Binary Search Trees (BST), Heaps, and AVL trees. Unlike arrays or linked lists which are linear, trees are **non-linear**, allowing for efficient representation of hierarchical relationships.

## Development

### Core Concept
Each element in a binary tree is a **Node**. A node typically contains:
1.  **Data**: The value stored (e.g., int, string).
2.  **Left Pointer**: Reference to the left child node.
3.  **Right Pointer**: Reference to the right child node.

The top-most node is the **Root**. Nodes with no children are called **Leaves**.

### Tree Traversals
Traversing a tree means visiting every node exactly once.
1.  **Pre-Order** (Root, Left, Right): Used to copy a tree.
2.  **In-Order** (Left, Root, Right): Used in BSTs to get sorted values.
3.  **Post-Order** (Left, Right, Root): Used to delete a tree (delete children before parent).
4.  **Level-Order** (Breadth-First): Visit nodes level by level.

### Properties
- **Max Nodes**: At depth `d`, max nodes = $2^d$.
- **Height**: The number of edges from root to the deepest leaf.
- **Max Nodes**: A binary tree of height `h` has at most $2^{h+1} - 1$ nodes.

## Code Examples (Go)

### 1. Node Structure
```go
package main

import "fmt"

type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

// Helper to create a new node
func NewNode(val int) *TreeNode {
    return &TreeNode{Val: val}
}
```

### 2. Recursive Traversals
```go
// PreOrder: Root -> Left -> Right
func PreOrder(root *TreeNode) {
    if root == nil {
        return
    }
    fmt.Printf("%d ", root.Val)
    PreOrder(root.Left)
    PreOrder(root.Right)
}

// InOrder: Left -> Root -> Right
func InOrder(root *TreeNode) {
    if root == nil {
        return
    }
    InOrder(root.Left)
    fmt.Printf("%d ", root.Val)
    InOrder(root.Right)
}
```

### 3. Level Order Traversal (Using Queue)
```go
func LevelOrder(root *TreeNode) {
    if root == nil {
        return
    }
    queue := []*TreeNode{root}

    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:] // Dequeue

        fmt.Printf("%d ", curr.Val)

        if curr.Left != nil {
            queue = append(queue, curr.Left)
        }
        if curr.Right != nil {
            queue = append(queue, curr.Right)
        }
    }
}
```

## Go Application & Ecosystem
Go does not have a built-in generic Tree type in the standard library. Developers implement their own `struct` pointers as shown above. This flexibility allows embedding trees into any custom data structure.

- **Recursion**: Go handles recursion well, but deep recursion on massive trees might hit the stack limit (though Go's stack is dynamic and starts small, growing up to 1GB).
- **Concurrency**: You can process independent subtrees concurrently using Goroutines, but you must be careful with shared memory if modifying the tree.

## Interview Preparation

### Common Questions
1.  **Maximum Depth of Binary Tree**:
    -   *Logic*: `max(maxDepth(root.Left), maxDepth(root.Right)) + 1`
2.  **Invert Binary Tree** (Famous Google question):
    -   *Logic*: Swap left and right children recursively.
3.  **Diameter of Binary Tree**:
    -   *Logic*: Longest path between any two nodes. May not pass through root.
4.  **Symmetric Tree**:
    -   *Logic*: Check if tree is a mirror of itself (left.left == right.right).

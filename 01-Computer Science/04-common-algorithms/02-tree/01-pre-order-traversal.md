---
---

# Pre-Order Traversal

## Abstract
**Pre-Order Traversal** is a depth-first traversal method for trees where the current node is visited **before** its children. The order of operations is **Root $\rightarrow$ Left $\rightarrow$ Right**. It is commonly used to create a copy of a tree or to serialize a tree structure (prefix notation).

## Development

### Core Concept
1.  **Visit** the root node (e.g., print its value).
2.  Recursively traverse the **Left** subtree.
3.  Recursively traverse the **Right** subtree.

### Complexity
- **Time**: $O(n)$ (Visits every node once).
- **Space**: $O(h)$ where $h$ is height (recursion stack). Worst case $O(n)$ for skewed trees.

## Code Examples (Go)

### Recursive Implementation
```go
package main

import "fmt"

type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func PreOrder(root *TreeNode) {
    if root == nil {
        return
    }
    fmt.Printf("%d ", root.Val) // Visit Root
    PreOrder(root.Left)         // Traverse Left
    PreOrder(root.Right)        // Traverse Right
}
```

### Iterative Implementation (Using Stack)
To emulate recursion, we use an explicit stack. Note that we push **Right** child first so that **Left** is popped and processed first.

```go
func PreOrderIterative(root *TreeNode) {
    if root == nil {
        return
    }
    
    stack := []*TreeNode{root}
    
    for len(stack) > 0 {
        // Pop
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        fmt.Printf("%d ", node.Val)
        
        // Push Right first!
        if node.Right != nil {
            stack = append(stack, node.Right)
        }
        // Push Left second (so it's popped first)
        if node.Left != nil {
            stack = append(stack, node.Left)
        }
    }
}
```

## Go Application
- **Serialization**: Pre-order is excellent for serializing trees because the first element is always the root, making reconstruction easier.
- **Copying**: Generating a deep copy of a tree structure.

## Interview Preparation
1.  **Reconstruct Tree**: Can you reconstruct a binary tree from just Pre-Order traversal?
    -   *Answer*: No, you need In-Order + Pre-Order to uniquely identify the tree (unless it's a BST).
2.  **Expression Trees**: Pre-order traversal of an expression tree yields the **Prefix** (Polish) notation (e.g., `+ A B`).

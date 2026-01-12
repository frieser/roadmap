---
---

# In-Order Traversal

## Abstract
**In-Order Traversal** is a depth-first traversal method where the current node is visited **between** its children. The order is **Left $\rightarrow$ Root $\rightarrow$ Right**. Its most significant property is that for a **Binary Search Tree (BST)**, in-order traversal retrieves the keys in **sorted (non-decreasing) order**.

## Development

### Core Concept
1.  Recursively traverse the **Left** subtree.
2.  **Visit** the root node.
3.  Recursively traverse the **Right** subtree.

### Complexity
- **Time**: $O(n)$.
- **Space**: $O(h)$.

## Code Examples (Go)

### Recursive Implementation
```go
func InOrder(root *TreeNode) {
    if root == nil {
        return
    }
    InOrder(root.Left)          // Traverse Left
    fmt.Printf("%d ", root.Val) // Visit Root
    InOrder(root.Right)         // Traverse Right
}
```

### Iterative Implementation
The logic is slightly more complex than Pre-Order. We must go as left as possible before processing.

```go
func InOrderIterative(root *TreeNode) {
    var stack []*TreeNode
    curr := root
    
    for curr != nil || len(stack) > 0 {
        // Go deep left
        for curr != nil {
            stack = append(stack, curr)
            curr = curr.Left
        }
        
        // Pop and process
        curr = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        fmt.Printf("%d ", curr.Val)
        
        // Move to right subtree
        curr = curr.Right
    }
}
```

## Go Application
- **BST Validation**: Check if a tree is a valid BST by verifying the in-order sequence is strictly increasing.
- **Sorting**: "Tree Sort" builds a BST and then traverses in-order.

## Interview Preparation
1.  **Kth Smallest Element**: How to find the Kth smallest element in a BST?
    -   *Answer*: Do an in-order traversal and stop at the Kth visited node.
2.  **Successor/Predecessor**: In-order traversal helps find the immediate successor or predecessor of a node in a BST.

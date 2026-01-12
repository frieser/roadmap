---
---

# Post-Order Traversal

## Abstract
**Post-Order Traversal** is a depth-first traversal method where the current node is visited **after** its children. The order is **Left $\rightarrow$ Right $\rightarrow$ Root**. It is essential for operations where you must process children before the parent, such as deleting a tree (freeing memory) or evaluating mathematical expressions.

## Development

### Core Concept
1.  Recursively traverse the **Left** subtree.
2.  Recursively traverse the **Right** subtree.
3.  **Visit** the root node.

### Complexity
- **Time**: $O(n)$.
- **Space**: $O(h)$.

## Code Examples (Go)

### Recursive Implementation
```go
func PostOrder(root *TreeNode) {
    if root == nil {
        return
    }
    PostOrder(root.Left)        // Traverse Left
    PostOrder(root.Right)       // Traverse Right
    fmt.Printf("%d ", root.Val) // Visit Root
}
```

### Iterative Implementation (Two Stacks Trick)
A simple way to implement this iteratively is to do a modified Pre-Order (Root -> Right -> Left) and push the result to a second stack (or reverse the list) to get Left -> Right -> Root.

```go
func PostOrderIterative(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    
    stack := []*TreeNode{root}
    var result []int
    
    // Result will be Root -> Right -> Left (reverse of desired)
    for len(stack) > 0 {
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        result = append(result, node.Val)
        
        if node.Left != nil {
            stack = append(stack, node.Left)
        }
        if node.Right != nil {
            stack = append(stack, node.Right)
        }
    }
    
    // Reverse result to get Left -> Right -> Root
    for i, j := 0, len(result)-1; i < j; i, j = i+1, j-1 {
        result[i], result[j] = result[j], result[i]
    }
    return result
}
```

## Go Application
- **Tree Deletion**: In languages with manual memory management (C++), post-order is required to delete children before the parent. In Go (GC), this is less critical but logic still applies to resource cleanup.
- **Expression Evaluation**: Post-order traversal of an expression tree yields **Reverse Polish Notation (RPN)** (e.g., `A B +`).

## Interview Preparation
1.  **Lowest Common Ancestor**: Post-order logic is often used to bubble up boolean flags ("found p", "found q") from subtrees to find the LCA.
2.  **Max Depth/Height**: Calculating height requires knowing the height of children first ($1 + \max(left, right)$), which is implicitly post-order.

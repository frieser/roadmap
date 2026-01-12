---
---

# Pre-Order Traversal

## Summary
**Pre-Order Traversal** is a DFS variant where the nodes are visited in the order: **Root $\to$ Left Child $\to$ Right Child**.

## Key Property
Pre-Order traversal visits the parent before any of its children. This preserves the **structure** of the tree if used for serialization.

## Code Examples (Go)

```go
func PreOrder(node *Node) {
    if node == nil {
        return
    }
    
    fmt.Printf("%d ", node.Val) // 1. Visit Root
    PreOrder(node.Left)      // 2. Traverse Left
    PreOrder(node.Right)     // 3. Traverse Right
}
```

### Example Output
Tree:
```
    10
   /  \
  5    15
```
Output: `10, 5, 15`

## Use Cases
1.  **Copying a Tree**: You must create the root before you can attach children to it.
2.  **Serialization/Prefix Notation**: Storing a tree structure in a file/string so it can be reconstructed identically.

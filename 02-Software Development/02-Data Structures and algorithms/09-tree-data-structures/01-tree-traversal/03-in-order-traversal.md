---
---

# In-Order Traversal

## Summary
**In-Order Traversal** is a DFS variant where the nodes are visited in the order: **Left Child $\to$ Root $\to$ Right Child**.

## Key Property
In a **Binary Search Tree (BST)**, In-Order traversal retrieves the values in **Sorted (Non-Decreasing) Order**.

## Code Examples (Go)

```go
func InOrder(node *Node) {
    if node == nil {
        return
    }
    
    InOrder(node.Left)       // 1. Traverse Left
    fmt.Printf("%d ", node.Val) // 2. Visit Root
    InOrder(node.Right)      // 3. Traverse Right
}
```

### Example Output
Tree:
```
    10
   /  \
  5    15
```
Output: `5, 10, 15` (Sorted)

## Use Cases
1.  Getting sorted data from a BST.
2.  Flattening a BST into a sorted linked list.

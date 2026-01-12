---
---

# Post-Order Traversal

## Summary
**Post-Order Traversal** is a DFS variant where the nodes are visited in the order: **Left Child $\to$ Right Child $\to$ Root**.

## Key Property
Post-Order traversal visits children before their parent. This is essential for operations that depend on children being processed first (bottom-up).

## Code Examples (Go)

```go
func PostOrder(node *Node) {
    if node == nil {
        return
    }
    
    PostOrder(node.Left)     // 1. Traverse Left
    PostOrder(node.Right)    // 2. Traverse Right
    fmt.Printf("%d ", node.Val) // 3. Visit Root
}
```

### Example Output
Tree:
```
    10
   /  \
  5    15
```
Output: `5, 15, 10`

## Use Cases
1.  **Deleting a Tree**: You must delete children (free memory) before deleting the parent to avoid memory leaks (in non-GC languages).
2.  **Evaluating Math Expressions**: In an Expression Tree, you evaluate operands (children) before the operator (root).
    *   Ex: `(3 + 4)` -> Left(3), Right(4), Root(+) -> `3 4 +` (Reverse Polish Notation).

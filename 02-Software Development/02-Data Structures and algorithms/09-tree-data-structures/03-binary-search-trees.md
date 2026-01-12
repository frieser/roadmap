---
---

# Binary Search Trees (BST)

## Summary
A **Binary Search Tree (BST)** is a binary tree with a strict ordering property: for every node, all values in its **left** subtree are smaller, and all values in its **right** subtree are larger. This property allows for fast lookup, insertion, and deletion, similar to Binary Search on an array.

## Detailed Explanation

### Core Property
$$ \text{Left.Value} < \text{Root.Value} < \text{Right.Value} $$

### Operations
1.  **Search**: Start at root. If `target < current`, go left. If `target > current`, go right.
2.  **Insert**: Search for the value. When you hit `nil`, put the new node there.
3.  **Delete**:
    *   **Leaf**: Just remove.
    *   **One Child**: Replace node with its child.
    *   **Two Children**: Find the **In-Order Successor** (smallest in right subtree), replace value, and delete the successor.

### Complexity
| Type | Average | Worst Case | Notes |
| :--- | :--- | :--- | :--- |
| **Time** | $O(\log n)$ | $O(n)$ | Worst case is a "skewed" tree (linked list). |
| **Space** | $O(h)$ | $O(n)$ | Stack space for recursion. |

## Code Examples (Go)

```go
package main

type Node struct {
    Value int
    Left  *Node
    Right *Node
}

// Insert adds a value to the BST
func (n *Node) Insert(val int) *Node {
    if n == nil {
        return &Node{Value: val}
    }
    if val < n.Value {
        n.Left = n.Left.Insert(val)
    } else {
        n.Right = n.Right.Insert(val)
    }
    return n
}

// Search returns true if val exists
func (n *Node) Search(val int) bool {
    if n == nil {
        return false
    }
    if val < n.Value {
        return n.Left.Search(val)
    } else if val > n.Value {
        return n.Right.Search(val)
    }
    return true
}
```

## Go Application
Go doesn't have a built-in BST in the standard library because:
1.  `map` (Hash Table) covers most O(1) key-value needs.
2.  When range queries are needed, self-balancing trees (like B-Trees or AVL) are used via third-party libraries (e.g., `google/btree`).

## Interview Questions

**Q: What makes a BST degenerate into a Linked List?**
**A:** If you insert sorted data (e.g., 1, 2, 3, 4, 5) into a naive BST, every new node becomes the right child of the previous one. The height becomes $N$, degrading search to $O(N)$.

**Q: How do you validate if a Binary Tree is a BST?**
**A:** You cannot just check `Left < Node < Right` locally. You must track a valid range `(min, max)` for each node.
*   Root: `(-inf, +inf)`
*   Left Child: `(-inf, Root.Value)`
*   Right Child: `(Root.Value, +inf)`

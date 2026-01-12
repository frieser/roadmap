---
---

# Balanced Tree

## Abstract
A **Balanced Binary Tree** is a tree structure where the height of the left and right subtrees of any node differs by no more than 1 (or a small constant factor). This balancing ensures that the tree height remains logarithmic **O(log n)** relative to the number of nodes, preventing the tree from degenerating into a linked list and guaranteeing efficient search, insert, and delete operations.

## Development

### Why Balance Matters?
- **Perfect Balance**: Height is exactly $\lfloor \log_2 n \rfloor$. Hard to maintain during updates.
- **AVL / Red-Black**: Maintain "good enough" balance (height $\approx k \cdot \log n$).
    - **AVL Tree**: Strict balance (diff $\le 1$). Faster lookups, slower inserts (more rotations).
    - **Red-Black Tree**: Looser balance. Faster inserts, slightly slower lookups.

### Operations
Balancing is achieved via **Rotations** (Left Rotation, Right Rotation) performed after insertions or deletions that violate the balance property.

## Code Examples (Go)

### Check Balanced Property (O(n))
We calculate the height of each subtree and return -1 if unbalanced.

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func IsBalanced(root *TreeNode) bool {
    return checkHeight(root) != -1
}

func checkHeight(node *TreeNode) int {
    if node == nil {
        return 0
    }

    leftH := checkHeight(node.Left)
    if leftH == -1 {
        return -1
    }

    rightH := checkHeight(node.Right)
    if rightH == -1 {
        return -1
    }

    diff := leftH - rightH
    if diff < 0 {
        diff = -diff
    }

    if diff > 1 {
        return -1 // Not balanced
    }

    // Return max height + 1
    if leftH > rightH {
        return leftH + 1
    }
    return rightH + 1
}
```

## Go Application
- **Databases**: B-Trees (self-balancing) are used in database indexing (Postgres, MySQL).
- **Go Runtime**: The Go scheduler and memory allocator use balanced tree-like structures (Treaps or similar) internally for tracking memory spans.
- **Libraries**: `github.com/emirpasic/gods` provides Red-Black and AVL tree implementations since the standard library does not.

## Interview Preparation
1.  **How to rebalance a tree?**: Explain Left/Right rotations.
2.  **AVL vs Red-Black**: AVL for read-heavy, Red-Black for write-heavy.
3.  **Convert Sorted Array to Balanced BST**: Pick the middle element as root, recurse left and right. (O(n)).

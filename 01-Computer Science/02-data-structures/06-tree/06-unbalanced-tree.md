---
---

# Unbalanced Tree

## Abstract
An **Unbalanced Tree** (or Degenerate Tree) is a binary tree where the structure is skewed, often resembling a **Linked List**. This occurs when nodes are inserted in sorted (or nearly sorted) order into a basic BST without rebalancing mechanisms. In an unbalanced tree, the height approaches **N** (number of nodes), degrading operation complexity from O(log n) to **O(n)**.

## Development

### The Degenerate Case
Imagine inserting: `1, 2, 3, 4, 5` into a standard BST.
- 1 is Root.
- 2 is Right child of 1.
- 3 is Right child of 2...
- Structure: `1 -> 2 -> 3 -> 4 -> 5` (Right-skewed).

### Performance Impact
| Metric | Balanced Tree | Unbalanced (Worst Case) |
|--------|---------------|-------------------------|
| Height | $log_2 n$     | $n$                     |
| Search | $O(log n)$    | $O(n)$                  |
| Insert | $O(log n)$    | $O(n)$                  |

This performance degradation is why self-balancing trees (AVL, Red-Black) are critical for production systems.

## Code Examples (Go)

### Creating an Unbalanced Tree (Example)
```go
package main

import "fmt"

type TreeNode struct {
    Val   int
    Left, Right *TreeNode
}

// Simple insert without rebalancing
func Insert(root *TreeNode, val int) *TreeNode {
    if root == nil {
        return &TreeNode{Val: val}
    }
    if val < root.Val {
        root.Left = Insert(root.Left, val)
    } else {
        root.Right = Insert(root.Right, val)
    }
    return root
}

func main() {
    // Inserting sorted data creates a degenerate tree
    var root *TreeNode
    nums := []int{1, 2, 3, 4, 5}
    
    for _, n := range nums {
        root = Insert(root, n)
    }
    
    // Result is essentially a linked list:
    // 1
    //  \
    //   2
    //    \
    //     3...
}
```

## Go Application & Mitigation
In Go, if you need a tree structure that guarantees performance, do **not** use a naive BST implementation if the input data might be sorted.
- **Mitigation**: Use a randomized approach (Treap) or a library implementing Red-Black trees.
- **Alternative**: Use `map` (Hash Table) which handles collisions better and provides O(1) average case, avoiding the "sorted input" pitfall of trees.

## Interview Preparation
1.  **Worst case for BST Insertion?**: Sorted or reverse-sorted input.
2.  **How to fix an unbalanced tree?**:
    -   Perform an In-Order traversal to get a sorted array.
    -   Rebuild the tree using the "middle element as root" strategy recursively.
    -   Or apply rotations (Day-Stout-Warren algorithm).
3.  **Time Complexity**: Be sure to distinguish between "Average Case" O(log n) and "Worst Case" O(n) when discussing BSTs.

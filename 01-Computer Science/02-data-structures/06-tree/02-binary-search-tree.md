---
---

# Binary Search Tree (BST)

## Abstract
A **Binary Search Tree (BST)** is a specific type of binary tree that maintains a sorted order property. For every node, all keys in the **left subtree** are less than the node's key, and all keys in the **right subtree** are greater than the node's key. This property enables efficient **searching, insertion, and deletion** operations, typically in **O(log n)** time.

## Development

### Core Concept
The BST property must hold for **every** node in the tree, not just the root.

$$ \forall n \in Nodes: Left.Key < n.Key < Right.Key $$

### Operations Complexity
| Operation | Average Case | Worst Case (Skewed) |
|-----------|--------------|---------------------|
| Search    | O(log n)     | O(n)                |
| Insert    | O(log n)     | O(n)                |
| Delete    | O(log n)     | O(n)                |

The worst case occurs when the tree becomes a "linked list" (e.g., inserting 1, 2, 3, 4, 5 in order). Self-balancing trees (AVL, Red-Black) solve this.

## Code Examples (Go)

### 1. Search
```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func Search(root *TreeNode, val int) *TreeNode {
    if root == nil || root.Val == val {
        return root
    }
    if val < root.Val {
        return Search(root.Left, val)
    }
    return Search(root.Right, val)
}
```

### 2. Insert
```go
func Insert(root *TreeNode, val int) *TreeNode {
    if root == nil {
        return &TreeNode{Val: val}
    }
    if val < root.Val {
        root.Left = Insert(root.Left, val)
    } else if val > root.Val {
        root.Right = Insert(root.Right, val)
    }
    return root
}
```

### 3. Validation (Check if BST)
Common interview mistake: Checking only `Left.Val < Root.Val` is insufficient. You must track min/max constraints down the tree.

```go
func IsValidBST(root *TreeNode) bool {
    return validate(root, nil, nil)
}

func validate(node *TreeNode, min, max *int) bool {
    if node == nil {
        return true
    }
    if (min != nil && node.Val <= *min) || (max != nil && node.Val >= *max) {
        return false
    }
    return validate(node.Left, min, &node.Val) && 
           validate(node.Right, &node.Val, max)
}
```

## Go Application & Ecosystem
BSTs are not in the Go standard library (which prefers `map` for key-value stores). However, the concepts are critical for:
- **Database Indexes**: Understanding B-Trees (a generalization of BST).
- **Router Logic**: Prefix trees (Tries) used in HTTP routers (like `gin` or `chi`) share similar traversal logic.

## Interview Preparation

### Common Questions
1.  **Validate Binary Search Tree**: (Code above).
2.  **Lowest Common Ancestor (LCA) in BST**:
    -   *Logic*: If both p and q are smaller than root, go left. If both bigger, go right. Otherwise, root is the split point (LCA).
3.  **Kth Smallest Element**:
    -   *Logic*: In-order traversal gives sorted elements. Stop at the Kth element.
4.  **Delete Node in BST**:
    -   *Logic*: If node has 0 children (remove), 1 child (replace), 2 children (find inorder successor, replace value, delete successor).

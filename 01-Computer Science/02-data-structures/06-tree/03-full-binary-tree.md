---
---

# Full Binary Tree

## Abstract
A **Full Binary Tree** (also known as a Proper or Plane Binary Tree) is a tree in which every node has either **0 or 2 children**. No node has exactly one child. This property makes the tree structurally efficient and is a key concept in understanding space complexity and compression algorithms like Huffman coding.

## Development

### Core Concept
- **Definition**: $\forall node \in Tree, degree(node) \in \{0, 2\}$.
- **Leaves**: Nodes with 0 children.
- **Internal Nodes**: Nodes with 2 children.

### Mathematical Properties
Let $L$ be the number of leaf nodes and $I$ be the number of internal nodes.
- $L = I + 1$
- Total nodes $N = 2I + 1$
- This implies a full binary tree always has an **odd number of nodes**.

### Usage
- **Huffman Coding**: The tree built for Huffman encoding is always a full binary tree.
- **Expression Trees**: Represents mathematical expressions (leaves are operands, internal nodes are operators).

## Code Examples (Go)

### Check if Full Binary Tree
```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func IsFullBinaryTree(root *TreeNode) bool {
    // Empty tree is full
    if root == nil {
        return true
    }

    // Leaf node (0 children) -> True
    if root.Left == nil && root.Right == nil {
        return true
    }

    // If both children exist, recurse
    if root.Left != nil && root.Right != nil {
        return IsFullBinaryTree(root.Left) && IsFullBinaryTree(root.Right)
    }

    // If one child is nil and the other is not -> False
    return false
}
```

## Go Application
While less common as a standalone general-purpose structure, Full Binary Trees appear in specific algorithm implementations (compression, parsing). In Go, you'll rarely enforce this constraint unless building a specific tool like a Huffman compressor.

## Interview Preparation
1.  **Relationship between Leaves and Internal Nodes**: Prove $L = I + 1$.
2.  **Convert general tree to Full Binary Tree**: Not always possible without adding dummy nodes.
3.  **Identify structure**: Given a serialized array, determine if it forms a full binary tree.

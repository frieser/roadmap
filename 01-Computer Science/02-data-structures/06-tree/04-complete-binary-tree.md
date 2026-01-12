---
---

# Complete Binary Tree

## Abstract
A **Complete Binary Tree** is a binary tree in which every level, except possibly the last, is **completely filled**, and all nodes in the last level are as far **left** as possible. This structural guarantee allows complete binary trees to be efficiently represented as **arrays** without gaps, making them the foundation for **Heaps**.

## Development

### Core Concept
1.  **Filled Levels**: Levels $0$ to $h-1$ are full ($2^i$ nodes).
2.  **Left-Justified**: Level $h$ is filled from left to right.

### Array Representation (Crucial)
Because there are no gaps, we can map node indices:
- Node index: $i$
- Left Child: $2i + 1$
- Right Child: $2i + 2$
- Parent: $(i-1)/2$

This is how `container/heap` in Go (and heaps in general) avoids pointer overhead.

## Code Examples (Go)

### Check Completeness (BFS Approach)
We use a queue. If we encounter a `nil` node, we must not encounter any non-nil node afterwards.

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func IsCompleteTree(root *TreeNode) bool {
    if root == nil {
        return true
    }

    queue := []*TreeNode{root}
    seenNull := false

    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:] // Dequeue

        if curr == nil {
            seenNull = true
        } else {
            if seenNull {
                // We saw a null before, but now we see a node.
                // This means there's a gap or it's not left-aligned.
                return false
            }
            queue = append(queue, curr.Left)
            queue = append(queue, curr.Right)
        }
    }
    return true
}
```

## Go Application
- **Heaps**: The most common application. Go's `container/heap` assumes the underlying data is a slice representing a complete binary tree.
- **Segment Trees**: Often implemented as complete binary trees for range query optimization.

## Interview Preparation
1.  **Count Nodes in Complete Binary Tree**:
    -   *Naive*: O(n).
    -   *Optimal*: **O(log² n)**. Use the height properties. If left height == right height, left subtree is full ($2^h - 1$).
2.  **Why use arrays for heaps?**: Because completeness guarantees no wasted memory (gaps) in the array.

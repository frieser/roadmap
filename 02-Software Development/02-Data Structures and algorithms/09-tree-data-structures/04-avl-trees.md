---
---

# AVL Trees

## Summary
An **AVL Tree** (Adelson-Velsky and Landis) is a **Self-Balancing Binary Search Tree**. It maintains the BST property while ensuring that the height difference (balance factor) between left and right subtrees of any node is at most 1. This guarantees **$O(\log n)$** time complexity for all operations, preventing the "skewed tree" worst-case scenario.

## Detailed Explanation

### Balance Factor
$$ \text{Balance} = \text{Height(Left)} - \text{Height(Right)} $$
Allowed values: $\{-1, 0, 1\}$.

### Rotations
If an insertion or deletion causes the Balance Factor to become $\pm 2$, the tree performs **rotations** to restore balance:
1.  **Single Right (LL Case)**: Left child is too heavy.
2.  **Single Left (RR Case)**: Right child is too heavy.
3.  **Left-Right (LR Case)**: Left child has a heavy right subtree.
4.  **Right-Left (RL Case)**: Right child has a heavy left subtree.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Search** | $O(\log n)$ | Strictly balanced. |
| **Insert** | $O(\log n)$ | May require rebalancing (rotations). |
| **Delete** | $O(\log n)$ | May require rebalancing. |

## Go Application
Implementing an AVL tree from scratch is complex and rarely done in production application code. Use cases requiring ordered data and range queries typically use **B-Trees** or **Skip Lists** in Go ecosystem.

### Conceptual Structure (Go)
```go
type AVLNode struct {
    Value  int
    Height int // Extra field to track height
    Left   *AVLNode
    Right  *AVLNode
}
```

## Interview Questions

**Q: Why use an AVL tree over a Red-Black tree?**
**A:** AVL trees are **more strictly balanced** than Red-Black trees. This makes AVL trees **faster for lookups** (closer to optimal height) but **slower for insertions/deletions** (more frequent rotations). Use AVL for read-heavy workloads.

**Q: What is the worst-case height of an AVL tree?**
**A:** roughly $1.44 \times \log_2 n$. It is strictly logarithmic.

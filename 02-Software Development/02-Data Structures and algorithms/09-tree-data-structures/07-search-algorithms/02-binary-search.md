---
---

# Binary Search

## Summary
**Binary Search** is an efficient search algorithm ($O(\log n)$) that works on **sorted** structures by repeatedly dividing the search interval in half. In the context of **Trees**, this is the fundamental principle behind **Binary Search Trees (BST)**.

## Detailed Explanation

### Mechanism (Array vs Tree)

| Feature | Sorted Array | Binary Search Tree (BST) |
| :--- | :--- | :--- |
| **Access** | Random Access (Index) | Pointer Chasing |
| **Division** | `mid = (low+high)/2` | `root` is the "mid" |
| **Left Side** | Indices `0` to `mid-1` | `root.Left` subtree |
| **Right Side** | Indices `mid+1` to `end` | `root.Right` subtree |

### Core Logic
1.  Compare `target` with `current` (mid/root).
2.  If `target == current`, found!
3.  If `target < current`, go **Left**.
4.  If `target > current`, go **Right**.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Avg)** | $O(\log n)$ | Balanced Tree or Array. |
| **Time (Worst)** | $O(n)$ | Degenerate Tree (Linked List). |
| **Space** | $O(1)$ (Iterative) | $O(h)$ (Recursive). |

## Code Examples (Go)

### 1. BST Search (The "Tree" Binary Search)
```go
type Node struct {
    Val   int
    Left  *Node
    Right *Node
}

func BinarySearchTree(root *Node, target int) *Node {
    // Base case: root is null or key is present at root
    if root == nil || root.Val == target {
        return root
    }

    // Key is smaller than root's key
    if target < root.Val {
        return BinarySearchTree(root.Left, target)
    }

    // Key is larger than root's key
    return BinarySearchTree(root.Right, target)
}
```

### 2. Array Binary Search (Reference)
```go
import "slices"

func main() {
    items := []int{1, 3, 5, 8, 9}
    // Works exactly like searching a balanced BST
    idx, found := slices.BinarySearch(items, 5)
}
```

## Go Application
*   **Database Indexes**: B-Trees allow Binary Search-like efficiency on disk.
*   **In-Memory**: `slices.BinarySearch` is used for sorted slices.

## Interview Questions

**Q: Why is Binary Search on an Array faster than on a BST in practice?**
**A:** **Cache Locality**. An array stores elements contiguously in memory, maximizing CPU cache hits. A BST stores nodes scattered in the heap, causing cache misses ("pointer chasing"), even if the algorithmic complexity $O(\log n)$ is the same.

**Q: Can we perform Binary Search on a generic Binary Tree?**
**A:** No. The tree **must** possess the **BST Property** (Left < Node < Right). Without this ordering, there is no way to know which half to discard, forcing a Linear Search ($O(n)$).

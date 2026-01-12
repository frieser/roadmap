---
---

# Linear Search

## Summary
**Linear Search** is the most basic search algorithm. It sequentially checks every element in a collection until a match is found or the whole collection has been searched. While typically associated with arrays, in the context of **Trees**, Linear Search corresponds to a full **Tree Traversal** (like BFS or DFS).

## Detailed Explanation

### Mechanism (General)
1.  Start at the first item.
2.  Compare with the target.
3.  If match, return success.
4.  If not, move to next.
5.  Repeat until end.

### Mechanism (Tree Context)
Since Trees are non-linear, "Linear Searching" a tree means visiting every node because the data is **unsorted** (or the sorting property doesn't help with the specific query).
*   **DFS (Depth-First Search)**: Linear search diving deep first.
*   **BFS (Breadth-First Search)**: Linear search layer by layer.

### Complexity
| Type | Complexity | Notes |
| :--- | :--- | :--- |
| **Time (Worst)** | $O(n)$ | Must visit every node/element. |
| **Space** | $O(1)$ (Array) / $O(h)$ (Tree) | Recursion stack or Queue space needed for Trees. |

## Code Examples (Go)

### 1. Array Linear Search
```go
func LinearSearchArray(arr []int, target int) int {
    for i, v := range arr {
        if v == target {
            return i
        }
    }
    return -1
}
```

### 2. Tree Linear Search (DFS)
If the tree is **not** a BST, we must check every node.

```go
type Node struct {
    Val   int
    Left  *Node
    Right *Node
}

func SearchTree(root *Node, target int) *Node {
    if root == nil {
        return nil
    }
    // Check current
    if root.Val == target {
        return root
    }
    // Search Left
    if res := SearchTree(root.Left, target); res != nil {
        return res
    }
    // Search Right
    return SearchTree(root.Right, target)
}
```

## Interview Questions

**Q: If you have a Binary Search Tree (BST), would you use Linear Search?**
**A:** No. You would use **Binary Search** ($O(\log n)$) properties. Linear Search ($O(n)$) is only necessary if the tree is unsorted or if you are searching for non-indexed attributes.

**Q: What is the worst-case time complexity of searching an unsorted Binary Tree?**
**A:** $O(n)$. You might have to visit every single node, effectively performing a linear search.

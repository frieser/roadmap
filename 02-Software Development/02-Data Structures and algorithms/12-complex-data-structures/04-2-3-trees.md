---
---

# 2-3 Trees

## Summary
A **2-3 Tree** is a self-balancing search tree where every node can have **2 or 3 children** (and thus 1 or 2 keys). It is a specific type of B-Tree (Order 3). It was a precursor to the more general B-Tree and Red-Black Tree.

## Detailed Explanation

### Node Types
1.  **2-Node**: Has 1 key and 2 children. (Like a binary node).
    *   Left Child < Key < Right Child.
2.  **3-Node**: Has 2 keys ($k_1, k_2$) and 3 children.
    *   Left < $k_1$ < Middle < $k_2$ < Right.

### Properties
*   **Perfect Balance**: All leaf nodes are at the same depth.
*   **Growth**: The tree grows **upwards**. When a root node splits, the height increases by 1.

### Relationship to Red-Black Trees
A **Red-Black Tree** is essentially a binary representation of a 2-3-4 Tree (or 2-3 Tree).
*   A "3-Node" in a 2-3 Tree is represented as a Black node with a Red child in a Red-Black Tree.

## Complexity
| Operation | Time | Space |
| :--- | :--- | :--- |
| **Search** | $O(\log n)$ | $O(n)$ |
| **Insert** | $O(\log n)$ | $O(n)$ |
| **Delete** | $O(\log n)$ | $O(n)$ |

## Go Application
2-3 Trees are rarely implemented directly. They are primarily a **conceptual stepping stone** to understanding:
1.  **B-Trees**: A 2-3 Tree is just a B-Tree where $t=2$.
2.  **Red-Black Trees**: The balancing logic of RB trees is derived from 2-3-4 tree splitting rules.

## Interview Questions

**Q: How does a 2-3 Tree stay balanced?**
**A:** When a node overflows (gets 3 keys), it splits. The middle key moves **up** to the parent. If the parent overflows, it splits too. If the root splits, a new root is created, increasing height. This ensures all leaves remain at the same level.

**Q: Why use Red-Black trees instead of 2-3 trees?**
**A:** 2-3 Trees require variable-sized nodes (some hold 1 key, some 2). This complicates implementation and memory management. Red-Black trees use standard binary nodes (fixed size) with a simple color bit, making them easier to implement in languages like C/Go.

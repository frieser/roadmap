---
---

# Skip List

## Summary
A **Skip List** is a probabilistic data structure that allows $O(\log n)$ search complexity within an ordered sequence of elements. It is essentially a **Linked List with fast lanes**. It provides a simpler alternative to balanced trees (like AVL or Red-Black trees) while offering similar performance.

## Detailed Explanation

### Mechanism
*   **Layer 0**: A standard sorted linked list containing all elements.
*   **Layer 1**: Skips some elements from Layer 0 (express lane).
*   **Layer k**: Skips elements from Layer $k-1$.
*   **Top Layer**: Contains very few elements (fastest lane).

### Operations
*   **Search**: Start at the top layer. Move right until the next node is larger than the target. Then drop down to the next layer and repeat.
*   **Insert**: Insert into Layer 0. Then, flip a coin (probability $p=0.5$). If Heads, promote the node to Layer 1. Repeat until Tails.

### Complexity
| Type | Average | Worst Case | Space |
| :--- | :--- | :--- | :--- |
| **Search** | $O(\log n)$ | $O(n)$ | $O(n)$ |
| **Insert** | $O(\log n)$ | $O(n)$ | |
| **Delete** | $O(\log n)$ | $O(n)$ | |

*Note: Worst case is incredibly rare (like flipping "Heads" 100 times in a row).*

## Go Application
*   **Redis**: Sorted Sets (`zset`) are implemented using Skip Lists.
*   **LevelDB/RocksDB**: Memtables often use Skip Lists because they support concurrent writes better than rebalancing trees (locking a Skip List node is more localized than rotating a tree).

## Code Concept (Go)
```go
type Node struct {
    Val  int
    Next []*Node // Array of pointers for different levels
}
```

## Interview Questions

**Q: Skip List vs Balanced Tree (AVL/RB)?**
**A:**
*   **Simplicity**: Skip Lists are much easier to implement (no complex rotation logic).
*   **Concurrency**: Skip Lists are easier to make lock-free or fine-grained locked. Rebalancing a tree affects large portions of the structure, requiring broader locks.
*   **Space**: Skip Lists use slightly more space on average due to multiple forward pointers per node.

**Q: Why is it called "Probabilistic"?**
**A:** The height of an element is determined by a random process (coin flips) during insertion. We rely on probability to ensure the tree remains "balanced" (approx. logarithmic height) rather than enforcing strict structural rules.

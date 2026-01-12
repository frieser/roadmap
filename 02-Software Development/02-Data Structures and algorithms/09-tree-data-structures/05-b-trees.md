---
---

# B-Trees

## Summary
A **B-Tree** is a self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in logarithmic time. Unlike binary trees, B-Trees are designed for systems that read and write large blocks of data, such as **Databases** and **File Systems**.

## Detailed Explanation

### Key Characteristics
1.  **Multi-way**: A node can have more than 2 children (typically hundreds).
2.  **Fat Nodes**: Each node contains multiple keys (e.g., a node might hold keys `[10, 20, 30]` and pointers to children for intervals `<10`, `10-20`, `20-30`, `>30`).
3.  **Disk Optimization**: Node size is usually tuned to match the underlying disk page size (e.g., 4KB). This minimizes the number of disk I/O operations required to reach a leaf.
4.  **Balance**: All leaf nodes are at the same depth.

### B-Tree vs. B+ Tree
*   **B-Tree**: Data is stored in internal nodes and leaf nodes.
*   **B+ Tree**: Data is stored **only in leaf nodes**. Internal nodes are just guideposts. Leaves are linked together in a linked list for fast range scans. (Most DBs use B+ Trees).

### Complexity
| Operation | Time | Disk I/O |
| :--- | :--- | :--- |
| **Search** | $O(\log n)$ | $O(\log_B n)$ |
| **Insert** | $O(\log n)$ | $O(\log_B n)$ |

Where $B$ is the branching factor (e.g., 100). $\log_{100} n$ is extremely small.

## Go Application
Go does not have a B-Tree in the standard library, but it is the backbone of many Go-based databases (like **Etcd**, **CockroachDB**, **InfluxDB**).

*   **Library**: `github.com/google/btree` is the industry standard implementation in Go.

```go
package main

import (
    "fmt"
    "github.com/google/btree"
)

type Item int

func (i Item) Less(than btree.Item) bool {
    return i < than.(Item)
}

func main() {
    // Degree 2 (2-3-4 tree)
    tr := btree.New(2)
    
    tr.ReplaceOrInsert(Item(10))
    tr.ReplaceOrInsert(Item(20))
    
    fmt.Println("Has 10?", tr.Has(Item(10)))
}
```

## Interview Questions

**Q: Why are B-Trees used in Databases instead of AVL/Red-Black trees?**
**A:** Binary trees (AVL/RB) require jumping to a new random memory location (pointer) for every decision (left/right). In a database stored on disk, this jumping causes expensive Disk Seeks. B-Trees pack hundreds of keys into a single node (Disk Page), meaning we can decide among 100 branches with just **one** disk read.

**Q: What is the height of a B-Tree with millions of items?**
**A:** Very low, typically 3 or 4. This means finding any record takes only 3-4 disk reads.

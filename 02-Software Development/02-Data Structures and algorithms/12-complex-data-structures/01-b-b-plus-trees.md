---
---

# B-Trees and B+ Trees

## Summary
**B-Trees** and **B+ Trees** are self-balancing tree data structures designed for systems that read and write large blocks of data (like **Databases** and **File Systems**). They are optimized to minimize disk I/O operations by storing a large number of keys in a single node.

## Detailed Explanation

### 1. B-Tree (General)
*   **Properties**:
    *   Nodes can have multiple children (branching factor $B$, often > 100).
    *   Keys and Data are stored in **both** internal nodes and leaf nodes.
    *   All leaf nodes are at the same depth.
*   **Pros**: Efficient for random access if the key happens to be near the root.

### 2. B+ Tree (Database Standard)
A variation of the B-Tree used by almost all relational databases (MySQL/InnoDB, PostgreSQL).
*   **Properties**:
    *   **Internal Nodes**: Store **only keys** (guideposts), no data.
    *   **Leaf Nodes**: Store **all data** (or pointers to data records).
    *   **Linked Leaves**: All leaf nodes are linked in a **doubly linked list**.
*   **Pros**:
    *   **Range Scans**: Since leaves are linked, you find the start of the range ($O(\log n)$) and then scan sequentially ($O(k)$).
    *   **Density**: Internal nodes can pack more keys because they don't store data, resulting in a flatter tree (fewer disk seeks).

### Complexity
| Operation | Time | Disk I/O |
| :--- | :--- | :--- |
| **Search** | $O(\log n)$ | $O(\log_B n)$ |
| **Insert** | $O(\log n)$ | $O(\log_B n)$ |
| **Delete** | $O(\log n)$ | $O(\log_B n)$ |
| **Range Scan** | $O(\log n + k)$ | Very Low (Sequential Read) |

## Go Application
*   **Databases**: `etcd` uses `bbolt` (a B+ Tree implementation in Go).
*   **Libraries**: `go.etcd.io/bbolt` is a popular pure-Go key/value store based on B+ Trees.

## Interview Questions

**Q: Why do Databases use B+ Trees instead of B-Trees?**
**A:**
1.  **Range Queries**: B+ Trees allow scanning sequential data (e.g., `SELECT * FROM Users WHERE Age > 20`) efficiently by following the linked list at the leaves. B-Trees would require traversing up and down the tree.
2.  **Fan-out**: Since internal nodes don't store data, they can hold more keys. This increases the branching factor ($B$), reducing the tree height and the number of disk I/Os.

**Q: What is the "Page Size" relation?**
**A:** B-Tree nodes are typically sized to match the OS/Disk Page Size (e.g., 4KB or 16KB). This ensures that reading a node takes exactly one disk I/O operation.

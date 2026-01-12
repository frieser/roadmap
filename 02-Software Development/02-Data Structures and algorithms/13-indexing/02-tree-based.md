---
---

# Tree-Based Indexing

## Summary
**Tree-Based Indexing** organizes keys in a hierarchical structure to minimize disk I/O. It allows for logarithmic time complexity ($O(\log n)$) for searches, insertions, and deletions. The standard for general-purpose databases is the **B+ Tree**, while specialized trees exist for spatial and multidimensional data.

## Detailed Explanation

### 1. B+ Tree (The Standard)
*   **Usage**: Primary Keys, Foreign Keys, Range Queries.
*   **Mechanism**: Balanced Multi-way Search Tree.
*   **Why**: Optimizes for Disk Page Size. Reduces tree height to 3-4 levels for billions of rows.
*   **Range Query**: Very fast due to linked leaf nodes.

### 2. R-Tree (Spatial Indexing)
Used for multi-dimensional data like **Coordinates (X, Y)**, Maps, and CAD.
*   **Problem**: B-Trees sort in 1D ($A < B$). How do you sort 2D points?
*   **Solution**: Group nearby objects into **Minimum Bounding Rectangles (MBR)**.
*   **Search**: "Find all restaurants within 5km". The search checks intersecting rectangles recursively.
*   **Databases**: PostGIS (PostgreSQL), MongoDB (`2dsphere`).

### 3. LSM Tree (Log-Structured Merge-tree)
Used in write-heavy NoSQL databases (Cassandra, RocksDB, LevelDB).
*   **Mechanism**: Writes go to an in-memory tree (MemTable). When full, flushed to disk as an immutable Sorted String Table (SSTable).
*   **Pros**: Writes are Append-Only (Sequential I/O = Fast).
*   **Cons**: Reads might need to check multiple SSTables.

## Go Application
*   **R-Tree**: `github.com/dhconnelly/rtreego` is a popular library for spatial indexing in Go.
*   **LSM**: **BadgerDB** is a pure Go key-value store based on LSM trees.

## Interview Questions

**Q: Why don't we use Binary Search Trees (BST) for database indexing?**
**A:**
1.  **Height**: BSTs are deep ($O(\log_2 N)$). A B-Tree with branching factor 100 is shallow ($O(\log_{100} N)$).
2.  **Locality**: BST nodes are scattered in memory/disk. B-Tree nodes pack keys into pages, maximizing what you get from a single Disk I/O.

**Q: How does an R-Tree handle overlapping rectangles?**
**A:** Unlike B-Trees where a key belongs to exactly one path, R-Tree rectangles can overlap. A search might need to traverse multiple branches if the query area intersects with multiple bounding boxes. This makes worst-case search slower than B-Trees.

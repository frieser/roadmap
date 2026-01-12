---
---

# Linear Indexing

## Summary
**Linear Indexing** refers to the simplest form of data organization where records are stored sequentially or accessed via a simple mapping table. It serves as the baseline against which more complex indexing methods (like Trees or Hashes) are compared.

## Detailed Explanation

### 1. Full Table Scan (No Index)
If no index exists, finding a record requires a **Linear Search** ($O(N)$).
*   **Mechanism**: The database reads every block from disk, checks every row, and returns matches.
*   **Pros**: Zero maintenance overhead on inserts. Fastest for very small tables.
*   **Cons**: Unusable performance for large datasets ($> 10^6$ rows).

### 2. Dense Index
A file containing a (Key, Pointer) pair for **every** record in the data file.
*   **Structure**: Sorted by Key.
*   **Search**: Binary Search on the index file ($O(\log N)$) $\to$ Direct Pointer to Data ($O(1)$).
*   **Update Cost**: High. Inserting a record requires shifting elements in the index file to maintain order.

### 3. Sparse Index
A file containing (Key, Pointer) pairs for only **some** records (e.g., the first record of each disk block).
*   **Search**: Find the largest key $\le$ Target in index. Go to that block. Linear search the block.
*   **Pros**: Index is much smaller (fits in RAM).
*   **Cons**: Slightly slower search than Dense Index.

## Go Application
In simple Go applications using flat files (like CSVs or Logs):
*   **Log Files**: Are linearly indexed by time (append-only).
*   **`sort.Search`**: If you load a CSV into a slice and sort it, you are essentially creating an in-memory Dense Index.

## Interview Questions

**Q: When is a Full Table Scan better than an Index Scan?**
**A:**
1.  **Small Tables**: If the table fits in a few disk pages, reading the whole table is faster than reading the Index Page + Data Page (random I/O).
2.  **Low Selectivity**: If the query returns 90% of the rows (e.g., `WHERE Gender='M'`), using an index causes random seeking for every row. Sequential scan is faster due to I/O throughput.

**Q: Difference between Dense and Sparse Index?**
**A:** Dense has an entry for *every* record. Sparse has an entry for *blocks* of records. Dense is faster for search but larger and harder to update. Sparse is smaller but requires a mini-scan inside the block.

---
---

# ISAM (Indexed Sequential Access Method)

## Summary
**ISAM** is a file management system developed by IBM that allows records to be accessed either **sequentially** (in order) or **randomly** (via an index). It is the predecessor to modern database indexing techniques like B+ Trees.

## Detailed Explanation

### Structure
ISAM consists of two files:
1.  **Data File**: Records stored sequentially (sorted by Primary Key).
2.  **Index File**: A sparse index containing keys and pointers to blocks in the data file.

### Mechanism
*   **Static Index**: Unlike B-Trees, the ISAM index structure is **static**. It is built once.
*   **Overflow Areas**: If new records are added and a block is full, they are placed in a separate "overflow area".
*   **Degradation**: As the overflow area grows, performance degrades significantly. The file must be periodically reorganized (re-indexed).

### Complexity
*   **Initial Search**: Fast (Binary search on index + Direct access).
*   **Degraded Search**: Slow (Linear search through overflow chains).

## Go Application
Modern Go applications rarely use raw ISAM. However, understanding it explains why **LSM Trees** (Log-Structured Merge-trees) and **B+ Trees** were invented: to handle dynamic updates without the performance cliff of ISAM's overflow chains.

## Interview Questions

**Q: What is the main difference between ISAM and B+ Trees?**
**A:**
*   **ISAM**: Static index. Does not handle inserts/updates well (uses overflow areas). Requires periodic maintenance/rebuilding.
*   **B+ Tree**: Dynamic index. Splits and merges nodes automatically to stay balanced. No overflow chains. Consistent performance.

**Q: Why is ISAM considered "Indexed Sequential"?**
**A:** Because it supports both access patterns efficiently (initially):
*   **Indexed**: Jump to a specific record using the index.
*   **Sequential**: Read the next record physically adjacent on disk (fast for batch processing).

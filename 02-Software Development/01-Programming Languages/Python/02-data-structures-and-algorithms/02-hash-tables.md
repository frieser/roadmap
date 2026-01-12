# Hash Tables

## Summary
In Python, Hash Tables are implemented as **Dictionaries** (`dict`) and **Sets** (`set`). They provide O(1) average time complexity for lookups, insertions, and deletions. Python's implementation is highly optimized, using **Open Addressing** with a specialized probing strategy.

## Detailed Explanation

### 1. Dictionary Implementation
Python dicts map keys to values.
*   **Hashing**: `hash(key)` generates a hash code.
*   **Index**: `hash % capacity` determines the position in the array.
*   **Collision Resolution**: **Open Addressing** (not Chaining). When a collision occurs, Python probes for the next empty slot using a pseudo-random permutation based on the hash (perturbation shift).

### 2. Compact Dictionaries (Python 3.6+)
Before 3.6, dicts were sparse arrays. Now, they are split into two:
1.  **Indices Array**: Sparse array storing only indices (8-bit integers usually).
2.  **Entries Array**: Dense array storing `[hash, key, value]` in insertion order.

**Benefit**:
*   **Memory**: Uses 20-25% less memory.
*   **Ordering**: Insertion order is preserved naturally.
*   **Iteration**: Faster iteration over the dense entries array.

### 3. Sets
Sets (`set`) are essentially dictionaries with only keys (no values). They share the same underlying hash table mechanism.

### Code Example: Hash Map Usage
```python
# Creating
counts = {}

# O(1) Insert
counts["apple"] = 1

# O(1) Lookup
if "apple" in counts:
    print(counts["apple"])

# O(1) Delete
del counts["apple"]
```

## Interview Questions

**Q: How does Python handle hash collisions?**
**A:** Python uses **Open Addressing** with a probing strategy. It doesn't use linked lists (chaining). If a slot is taken, it calculates a new index using a specialized formula (`idx = (5*idx + 1) + perturbation`) until an empty slot is found.

**Q: Why must dictionary keys be immutable?**
**A:** Because the hash value determines the location. If the object changes (mutable), its hash might change, making it impossible to find the object again in the table. Lists cannot be keys; tuples can.

**Q: Why are Python dictionaries ordered?**
**A:** Since Python 3.7 (implementation detail in 3.6), dicts use a "compact" representation where items are stored in a dense array in insertion order, and a separate sparse array maps hash indices to positions in that dense array.

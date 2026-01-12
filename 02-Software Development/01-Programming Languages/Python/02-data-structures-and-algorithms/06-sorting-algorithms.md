# Sorting Algorithms

## Summary
Python's built-in sorting (`list.sort()` and `sorted()`) uses **Timsort**, a hybrid algorithm derived from Merge Sort and Insertion Sort. It is designed to perform very well on real-world data, which often contains pre-sorted runs.

## Detailed Explanation

### Timsort (Python's Default)
*   **Complexity**:
    *   Best Case: O(n) (already sorted)
    *   Average/Worst: O(n log n)
    *   Space: O(n)
*   **Stability**: **Stable**. (Preserves relative order of equal elements).

#### How Timsort Works
1.  **Run Identification**: It scans the array for natural runs (sequences already ordered).
2.  **Insertion Sort**: Small runs (minrun size, usually 32-64) are sorted using Insertion Sort (very fast for small arrays).
3.  **Merge**: It merges these sorted runs using a modified Merge Sort strategy.
4.  **Galloping**: When merging, if one run wins consistently, it switches to "galloping mode" (binary search) to move chunks of data at once.

### `sort()` vs `sorted()`
*   **`list.sort()`**: In-place. Modifies the original list. Returns `None`. Faster (saves memory allocation).
*   **`sorted(iterable)`**: Out-of-place. Creates a new list. Works on any iterable (tuples, dicts).

### Custom Sorting
Using the `key` argument (Schwartzian transform).

```python
data = ["apple", "Banana", "cherry"]
# Case-insensitive sort
data.sort(key=str.lower)

# Sort complex objects
users.sort(key=lambda u: u.age)
```

## Interview Questions

**Q: What algorithm does Python use for sorting?**
**A:** Timsort. It's a hybrid of Merge Sort and Insertion Sort, optimized for real-world data runs. It is stable and has O(n log n) worst-case time complexity.

**Q: Is Python's sort stable?**
**A:** Yes. If two elements have equal keys, their original order is preserved. This allows for multi-pass sorting (e.g., sort by name, then by age).

**Q: Why use `list.sort()` over `sorted()`?**
**A:** `list.sort()` sorts the list in-place, meaning it doesn't require allocating memory for a new list. It's more memory efficient if you don't need the original order.

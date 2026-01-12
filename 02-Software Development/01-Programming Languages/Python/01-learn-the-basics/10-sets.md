# Sets

## Summary
Sets are unordered, unindexed collections of **unique** elements. Defined with curly braces `{}`. Useful for membership testing and mathematical set operations.

## Detailed Explanation

### Creating Sets
```python
myset = {"apple", "banana", "cherry"}
# Duplicates are automatically removed
myset.add("apple") 
print(len(myset)) # 3
```

### Set Operations
Efficient mathematical operations.

*   **Union (`|`)**: All items from both sets.
*   **Intersection (`&`)**: Items present in both.
*   **Difference (`-`)**: Items in first but not second.
*   **Symmetric Difference (`^`)**: Items in either, but not both.

```python
a = {1, 2, 3}
b = {3, 4, 5}

print(a | b) # {1, 2, 3, 4, 5}
print(a & b) # {3}
print(a - b) # {1, 2}
```

### Frozen Sets
Immutable version of a set: `frozenset([1, 2, 3])`. Can be used as a dictionary key.

## Interview Questions

**Q: Are sets ordered?**
**A:** No. Items in a set have no defined order, and you cannot access them by index.

**Q: How do you remove duplicates from a list?**
**A:** Convert it to a set and back: `list(set(my_list))`. Note that this loses the original order.

**Q: Can you store a list inside a set?**
**A:** No. Set elements must be hashable (immutable). Lists are mutable, so they cannot be elements of a set. Tuples can be, if they contain only immutable items.

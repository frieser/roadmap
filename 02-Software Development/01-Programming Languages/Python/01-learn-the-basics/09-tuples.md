# Tuples

## Summary
Tuples are ordered, **immutable** collections. Once created, their items cannot be changed. Defined with parentheses `()`. They are faster than lists and used for data that shouldn't change.

## Detailed Explanation

### Creation and Access
```python
thistuple = ("apple", "banana", "cherry")
single_item = ("apple",) # Note the comma!

# Access is same as list
print(thistuple[0])
```

### Immutability
You cannot add, remove, or change elements.
```python
x = (1, 2)
# x[0] = 3 # Raises TypeError
```

### Unpacking
Tuples allow concise variable assignment.
```python
fruits = ("apple", "banana", "cherry")
(green, yellow, red) = fruits

# Using asterisk for remaining items
(green, *rest) = fruits
print(rest) # ['banana', 'cherry']
```

## Interview Questions

**Q: What is the main difference between a list and a tuple?**
**A:** Mutability. Lists are mutable (can change); tuples are immutable (cannot change). Tuples are also slightly more memory efficient and hashable (can be used as dictionary keys if they contain only immutable types).

**Q: How do you create a tuple with one element?**
**A:** You must include a comma: `t = (1,)`. Without the comma, Python treats `(1)` as an integer in parentheses.

**Q: When should you use a tuple instead of a list?**
**A:** When the data should not change (write-protection), when iterating over fixed data (slight performance boost), or when using the data as a dictionary key.

# Lists

## Summary
Lists are ordered, mutable collections of items. They allow duplicate elements and can contain mixed data types. Defined with square brackets `[]`.

## Detailed Explanation

### Basic Operations
```python
fruits = ["apple", "banana", "cherry"]

# Access
print(fruits[0])  # apple
print(fruits[-1]) # cherry (last item)

# Slice
print(fruits[1:3]) # ['banana', 'cherry']
```

### Modifying Lists
```python
fruits.append("orange")    # Add to end
fruits.insert(1, "mango")  # Insert at index
fruits.remove("banana")    # Remove by value
pop_val = fruits.pop()     # Remove last item and return it
```

### List Comprehension
A concise way to create lists.

```python
# [expression for item in iterable if condition]
squares = [x**2 for x in range(10) if x % 2 == 0]
```

### Copying
Lists are reference types. `list2 = list1` does NOT copy the list; it references the same memory.
*   **Shallow Copy**: `list2 = list1.copy()` or `list2 = list1[:]`
*   **Deep Copy**: `import copy; list2 = copy.deepcopy(list1)`

## Interview Questions

**Q: What is the difference between `append()` and `extend()`?**
**A:** `append()` adds its argument as a single element to the end of the list. `extend()` iterates over its argument and adds each element to the list.

**Q: Is a list mutable?**
**A:** Yes. You can change, add, and remove elements after the list is created without creating a new object.

**Q: How does list comprehension work?**
**A:** It creates a new list by applying an expression to each item in an iterable. It's often faster and more readable than a `for` loop for simple transformations.

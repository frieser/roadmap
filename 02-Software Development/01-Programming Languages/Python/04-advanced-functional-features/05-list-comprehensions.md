# List Comprehensions

## Summary
List Comprehension offers a concise syntax to create lists based on existing lists. It is often more readable and faster than standard `for` loops because the iteration happens at C-level inside the interpreter.

## Detailed Explanation

### Syntax
`[expression for item in iterable if condition]`

### Examples

#### Basic
```python
fruits = ["apple", "banana", "cherry", "kiwi", "mango"]
newlist = [x for x in fruits if "a" in x]
# ['apple', 'banana', 'mango']
```

#### With Transformation
```python
# Convert to uppercase
upper = [x.upper() for x in fruits]
```

#### With `if/else`
The syntax changes slightly when adding an `else`. The condition moves to the beginning (ternary operator).

```python
# [expression_if_true if condition else expression_if_false for item in iterable]
parity = ["Even" if i % 2 == 0 else "Odd" for i in range(5)]
# ['Even', 'Odd', 'Even', 'Odd', 'Even']
```

#### Nested Comprehensions
You can flatten a matrix (list of lists).

```python
matrix = [[1, 2, 3], [4, 5, 6]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6]
```

## Interview Questions

**Q: Is list comprehension faster than a for loop?**
**A:** Generally yes. List comprehensions are optimized in C and avoid the overhead of Python's `append` method calls and interpreter loop steps. However, for very complex logic, a for loop might be more readable.

**Q: Can you use list comprehension to create a dictionary?**
**A:** Yes, that's called Dictionary Comprehension. Syntax: `{key: value for item in iterable}`. Example: `{x: x**2 for x in range(5)}`.

**Q: What happens if the list comprehension is extremely large?**
**A:** It will generate the entire list in memory immediately. If the list is too large to fit in RAM, you should use a **Generator Expression** instead (replace brackets `[]` with parentheses `()`).

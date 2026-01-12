# Type Casting

## Summary
Type casting is the process of converting a variable from one type to another. In Python, this is usually done with constructor functions like `int()`, `float()`, and `str()`.

## Detailed Explanation

### Implicit Casting
Python automatically converts types when it makes sense (e.g., adding an integer to a float).

```python
x = 1   # int
y = 2.5 # float
z = x + y # z is 3.5 (float) - Implicit conversion
```

### Explicit Casting
Forced conversion using built-in functions.

*   **`int()`**: Constructs an integer from a float (truncates) or string literal.
*   **`float()`**: Constructs a float from an integer or string.
*   **`str()`**: Converts objects to their string representation.

```python
# Int conversion
a = int(2.8)    # 2 (truncates decimal)
b = int("3")    # 3

# Float conversion
c = float(2)    # 2.0
d = float("4.2") # 4.2

# String conversion
e = str(2)      # "2"
f = str(3.0)    # "3.0"
```

### Pitfalls
*   Converting a string with non-numeric characters to int/float raises `ValueError`.
*   Floating point arithmetic can have precision issues.

```python
int("abc") # Raises ValueError
```

## Interview Questions

**Q: What happens if you cast `int(3.9)`?**
**A:** It truncates the decimal part and returns `3`. It does not round to the nearest integer.

**Q: Can you convert a list to a set? What happens?**
**A:** Yes, `set([1, 2, 2, 3])` converts the list to a set. This removes all duplicate elements, resulting in `{1, 2, 3}`.

**Q: How do you check if a string represents a digit before casting?**
**A:** Use the `.isdigit()` string method. `if s.isdigit(): int(s)`.

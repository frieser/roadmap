# Variables and Data Types

## Summary
Python is dynamically typed, meaning you don't declare the type of a variable. The type is determined at runtime based on the value assigned. Everything in Python is an object.

## Detailed Explanation

### Basic Data Types
*   **Integers (`int`)**: Whole numbers, arbitrary precision.
*   **Floats (`float`)**: Decimal numbers (double precision).
*   **Strings (`str`)**: Sequences of Unicode characters.
*   **Booleans (`bool`)**: `True` or `False`.
*   **NoneType (`None`)**: Represents the absence of a value.

### Dynamic Typing
Variables can change type if reassigned.

```python
x = 5       # x is an int
x = "Sally" # x is now a str
```

### Type Checking
Use `type()` to check a variable's type.

```python
print(type(5))       # <class 'int'>
print(type(3.14))    # <class 'float'>
```

### Modern Type Hinting (Python 3.10+)
While dynamic, you can add hints for better IDE support and documentation.

```python
name: str = "Alice"
age: int | float = 25  # Union type (3.10+)
```

### String Formatting (f-strings)
Introduced in Python 3.6, f-strings are the preferred way to format strings.

```python
name = "Bob"
age = 30
# Self-documenting expression (Python 3.8+)
print(f"{name=}, {age=}") # Output: name='Bob', age=30
```

## Interview Questions

**Q: What does "dynamically typed" mean in Python?**
**A:** It means types are checked at runtime, and variables do not have a fixed type. A variable can hold an integer at one moment and a string at the next.

**Q: How do you declare a constant in Python?**
**A:** Python doesn't have true constants. By convention, variables named in `ALL_CAPS` are treated as constants by developers, but the interpreter does not enforce immutability.

**Q: What is the difference between `is` and `==`?**
**A:** `==` checks for value equality (do they look the same?). `is` checks for reference equality (are they the exact same object in memory?).

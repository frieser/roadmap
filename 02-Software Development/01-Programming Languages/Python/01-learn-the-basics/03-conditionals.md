# Conditionals

## Summary
Python supports standard conditional logic using `if`, `elif`, and `else`. Python 3.10 introduced Structural Pattern Matching (`match`/`case`), offering a more powerful alternative to complex `if-elif` chains.

## Detailed Explanation

### Basic `if` Statement
Blocks are defined by indentation.

```python
age = 18

if age >= 18:
    print("Adult")
elif age > 12:
    print("Teen")
else:
    print("Child")
```

### Ternary Operator
One-line conditional assignment.

```python
status = "Adult" if age >= 18 else "Minor"
```

### Structural Pattern Matching (Python 3.10+)
The `match` statement compares a value against patterns. It is more than just a "switch" statement; it can unpack sequences and match types.

```python
# Matching literals
status = 404
match status:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case _:
        print("Unknown")

# Matching types and unpacking
point = (0, 10)
match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y-axis at {y}")
    case (x, 0):
        print(f"X-axis at {x}")
    case (x, y):
        print(f"Point at {x}, {y}")
```

### Truthy and Falsy Values
In Python, the following evaluate to `False`:
*   `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`.
Everything else is `True`.

## Interview Questions

**Q: Does Python have a `switch` statement?**
**A:** Before Python 3.10, no. Developers used dictionaries or `if-elif` chains. Since Python 3.10, the `match` statement provides similar but more powerful functionality (pattern matching).

**Q: What is a "truthy" value?**
**A:** A value that evaluates to `True` in a boolean context. For example, a non-empty list `[1, 2]` is truthy, while an empty list `[]` is falsy.

**Q: How does `match` differ from a traditional `switch`?**
**A:** `match` can deconstruct data structures (tuples, lists, objects) and bind variables inside the case pattern, whereas a traditional `switch` usually only matches equality of values.

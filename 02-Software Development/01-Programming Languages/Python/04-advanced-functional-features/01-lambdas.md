# Lambdas

## Summary
A lambda is a small anonymous function defined with the `lambda` keyword. It can take any number of arguments but can only have **one expression**.

## Detailed Explanation

### Syntax
`lambda arguments : expression`

The expression is executed and the result is returned. No `return` statement is needed.

### Basic Usage
```python
# Regular function
def add(x, y):
    return x + y

# Equivalent Lambda
add_lambda = lambda x, y: x + y

print(add_lambda(5, 3)) # 8
```

### Common Use Cases
Lambdas are best used when you need a short function for a short period of time, often as an argument to higher-order functions like `map`, `filter`, or `sort`.

#### 1. Sorting with Key
```python
users = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 20},
    {"name": "Charlie", "age": 30}
]

# Sort by age
users.sort(key=lambda x: x["age"])
```

#### 2. Functional Methods (Map/Filter)
```python
nums = [1, 2, 3, 4]
squared = list(map(lambda x: x**2, nums)) # [1, 4, 9, 16]
evens = list(filter(lambda x: x % 2 == 0, nums)) # [2, 4]
```

### Limitations
1.  **Single Expression**: Cannot contain statements (like `print`, `if/else` block, `for` loops). Only conditional expressions (`x if c else y`) are allowed.
2.  **Debugging**: Stack traces show `<lambda>`, making it harder to identify which function failed.
3.  **No Docstrings**: You cannot attach documentation to a lambda.

## Interview Questions

**Q: What is the difference between a def function and a lambda?**
**A:** `def` creates a named function with multiple statements, docstrings, and a full body. `lambda` creates an anonymous function limited to a single expression.

**Q: Can a lambda contain an if/else statement?**
**A:** No, but it can contain a conditional **expression** (ternary operator). Example: `lambda x: "Even" if x % 2 == 0 else "Odd"`.

**Q: Why does PEP 8 (Python Style Guide) discourage assigning lambdas to variables?**
**A:** Because it defeats the purpose of an anonymous function and hurts debugging. Instead of `f = lambda x: x*2`, you should write `def f(x): return x*2` so the function has a proper name in tracebacks.

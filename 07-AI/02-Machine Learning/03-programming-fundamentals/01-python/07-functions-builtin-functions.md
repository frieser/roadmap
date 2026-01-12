---
tags: ['ai', 'roadmap']
---

## Summary
Python functions are the primary building blocks for modular and reusable code. This note covers the definition and invocation of functions, handling various argument types (`*args`, `**kwargs`), and leveraging Python's powerful built-in functional tools like `lambda`, `map`, `filter`, and `reduce`. Understanding these is crucial for data manipulation and building scalable AI pipelines.

## Detailed Explanation

### Defining Functions
In Python, functions are defined using the `def` keyword. They can return values using `return` (returning `None` by default).

```python
def greet(name, greeting="Hello"):
    """Greets a person with a message."""
    return f"{greeting}, {name}!"

print(greet("Alice"))             # Hello, Alice!
print(greet("Bob", greeting="Hi")) # Hi, Bob!
```

### Arguments and Parameters
Python supports several ways to pass arguments:
1.  **Positional Arguments**: Matched by position.
2.  **Keyword Arguments**: Matched by parameter name.
3.  **Default Arguments**: Parameters with a default value if none is provided.

#### `*args` and `**kwargs`
These allow functions to accept an arbitrary number of arguments.
*   **`*args`**: Collects extra positional arguments as a **tuple**.
*   **`**kwargs`**: Collects extra keyword arguments as a **dictionary**.

```python
def flexible_function(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

flexible_function(1, 2, 3, a="apple", b="banana")
# Positional: (1, 2, 3)
# Keyword: {'a': 'apple', 'b': 'banana'}
```

### Lambda Functions
Lambdas are small, anonymous one-line functions defined with the `lambda` keyword. They are often used as arguments to higher-order functions.

```python
# lambda arguments: expression
square = lambda x: x ** 2
print(square(5))  # 25
```

### Higher-Order Built-in Functions
These functions take other functions as arguments, facilitating a functional programming style.

1.  **`map(func, iterable)`**: Applies `func` to every item in `iterable`.
    ```python
    nums = [1, 2, 3]
    squares = list(map(lambda x: x**2, nums)) # [1, 4, 9]
    ```

2.  **`filter(func, iterable)`**: Keeps items for which `func` returns `True`.
    ```python
    nums = [1, 2, 3, 4, 5]
    evens = list(filter(lambda x: x % 2 == 0, nums)) # [2, 4]
    ```

3.  **`reduce(func, iterable)`**: Reduces an iterable to a single value (requires `functools`).
    ```python
    from functools import reduce
    nums = [1, 2, 3, 4]
    product = reduce(lambda x, y: x * y, nums) # 24
    ```

### Essential Built-in Functions
| Function | Description |
| --- | --- |
| `len()` | Returns the length of an object. |
| `enumerate()` | Returns pairs of `(index, item)`. |
| `zip()` | Aggregates elements from multiple iterables. |
| `sorted()` | Returns a new sorted list from an iterable. |
| `any() / all()` | Returns `True` if any/all elements are truthy. |
| `range()` | Generates a sequence of numbers. |

## Interview Questions

**Q: What is the difference between `*args` and `**kwargs`?**
**A:** `*args` allows a function to accept any number of positional arguments, which are stored in a tuple. `**kwargs` allows a function to accept any number of keyword arguments, which are stored in a dictionary.

**Q: Why should you avoid using mutable default arguments (like `def func(a=[])`)?**
**A:** Default arguments in Python are evaluated only once at the time of function definition. If you use a mutable object like a list, it will be shared across all calls to the function, leading to unexpected behavior. Use `None` as the default instead.

**Q: What is the difference between `list.sort()` and `sorted(list)`?**
**A:** `list.sort()` is a method that sorts the list in-place and returns `None`. `sorted(list)` is a built-in function that returns a *new* sorted list, leaving the original list unchanged.

**Q: Explain how `zip()` works and what it returns.**
**A:** `zip()` takes multiple iterables and aggregates them into an iterator of tuples, where the i-th tuple contains the i-th element from each of the input iterables. It stops when the shortest iterable is exhausted.

**Q: When would you use a `lambda` function instead of a regular `def` function?**
**A:** Lambdas are used for simple, one-off logic, typically as arguments to functions like `map()`, `filter()`, or `sorted()`. For any complex logic, docstrings, or multiple statements, a regular `def` function is preferred for readability.

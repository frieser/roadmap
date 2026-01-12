# Functions

## Summary
Functions are defined using the `def` keyword. They organize code into reusable blocks. Python functions support default arguments, variable-length arguments, and can return multiple values (as tuples).

## Detailed Explanation

### Defining a Function

```python
def my_function(name="Guest"):
    return f"Hello, {name}"
```

### *args and **kwargs
To accept an arbitrary number of arguments:
*   `*args`: Tuple of positional arguments.
*   `**kwargs`: Dictionary of keyword arguments.

```python
def sum_all(*args):
    return sum(args)

def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")
```

### Lambda Functions
Small anonymous functions defined with `lambda`.

```python
add = lambda a, b: a + b
print(add(5, 3))
```

### Type Hinting (Annotations)
Standard since Python 3.5+.

```python
def greeting(name: str) -> str:
    return "Hello " + name
```

### Scope
Variables defined inside a function are local. To modify a global variable, use the `global` keyword.

## Interview Questions

**Q: What are `*args` and `**kwargs`?**
**A:** `*args` allows passing a variable number of positional arguments (received as a tuple). `**kwargs` allows passing a variable number of keyword arguments (received as a dictionary).

**Q: Can a function return multiple values?**
**A:** Yes, technically it returns a tuple containing the values. Python allows automatic unpacking of this tuple. `x, y = get_coordinates()`.

**Q: What is a lambda function?**
**A:** An anonymous, inline function defined with the `lambda` keyword. It can have any number of arguments but only one expression.

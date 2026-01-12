# Decorators

## Summary
A decorator is a function that takes another function and extends its behavior without explicitly modifying it. It uses the `@` syntax.

## Detailed Explanation

### Basic Structure
A decorator is a wrapper function. It typically accepts a function (`func`), defines a wrapper, and returns the wrapper.

```python
def my_decorator(func):
    def wrapper():
        print("Something before the function is called.")
        func()
        print("Something after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

### Decorating Functions with Arguments
The wrapper must accept `*args` and `**kwargs` to support any function signature.

```python
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

### Preserving Metadata (`functools.wraps`)
When you decorate a function, it loses its original name and docstring (it becomes the wrapper's). Use `@wraps` to fix this.

```python
from functools import wraps

def proper_decorator(func):
    @wraps(func) # Preserves name and docstring
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper
```

### Modern Typing (Python 3.10+)
Using `ParamSpec` allows type checkers to understand that the decorator preserves the signature of the input function.

```python
from typing import TypeVar, Callable, ParamSpec

P = ParamSpec("P")
R = TypeVar("R")

def typed_decorator(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print("Typed wrapper")
        return func(*args, **kwargs)
    return wrapper
```

## Interview Questions

**Q: What happens if you stack multiple decorators?**
**A:** They are applied from bottom to top (innermost to outermost).
```python
@decorator1
@decorator2
def func(): pass
```
Is equivalent to `func = decorator1(decorator2(func))`.

**Q: Why do we use `@functools.wraps`?**
**A:** Without it, the decorated function loses its metadata (like `__name__` and `__doc__`), taking on the metadata of the wrapper function instead. This confuses debugging tools and help() generation.

**Q: Can a decorator accept arguments?**
**A:** Yes. This requires a three-level function structure. The outermost function accepts the arguments and returns the actual decorator.

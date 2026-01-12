#Python
---
---

## Summary
The Python `typing` module, introduced in PEP 484, provides a framework for specifying type hints to enable static type checking. While Python remains a dynamically typed language and does not enforce these hints at runtime, they are invaluable for identifying bugs early, improving code readability, and enhancing IDE features like autocomplete. Modern Python versions (3.9+) have integrated many of these capabilities directly into built-in types, making the system more ergonomic.

## Detailed Explanation

### Type Hints Syntax
Type hints can be applied to variables, function parameters, and return values.
```python
# Variable annotation
age: int = 25

# Function annotation
def greet(name: str) -> str:
    return f"Hello, {name}"
```

### Core Types
The `typing` module provides several primitives for more complex types:
- **`list`, `dict`, `set`, `tuple`**: Since Python 3.9, you can use built-in collections with square brackets (e.g., `list[int]`).
- **`Optional[T]`**: Indicates that a value can be of type `T` or `None`.
- **`Union[T1, T2]`**: Indicates that a value can be either `T1` or `T2`. Python 3.10 introduced the pipe operator: `int | str`.
- **`Any`**: A special type indicating no type constraints; every type is compatible with `Any`.

```python
from typing import Optional, Union

def process_data(data: list[int], scale: Optional[float] = None) -> Union[int, str]:
    if scale:
        return int(sum(data) * scale)
    return "No scale provided"
```

### Generics and TypeVar
Generics allow you to write functions and classes that work with any type while maintaining type safety. `TypeVar` is used to create type variables.
```python
from typing import TypeVar, Sequence

T = TypeVar('T')  # Can be any type

def get_first_element(items: Sequence[T]) -> T:
    return items[0]

# usage
first_int = get_first_element([1, 2, 3])  # Inferred as int
first_str = get_first_element(["a", "b"]) # Inferred as str
```

### Callable
Used for annotating functions passed as arguments. Syntax: `Callable[[Arg1Type, Arg2Type], ReturnType]`.
```python
from typing import Callable

def apply_func(x: int, func: Callable[[int], int]) -> int:
    return func(x)

result = apply_func(5, lambda x: x * 2)
```

### Protocol (Structural Subtyping)
`Protocol` allows for "static duck typing." A class is considered a subtype of a Protocol if it implements the required methods, without needing explicit inheritance.
```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle:
    def draw(self) -> None:
        print("Drawing a circle")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())  # Works because Circle has a draw() method
```

## Interview Questions

**Q: Does Python enforce type hints at runtime?**
**A:** No. Type hints are ignored by the Python interpreter during execution. They are primarily used by static analysis tools (like mypy), IDEs, and linters to catch errors before the code runs.

**Q: What is the difference between `Any` and `object`?**
**A:** `object` is the root of the Python class hierarchy; every type is a subclass of `object`, but you can only perform operations supported by `object`. `Any` is a special "escape hatch" that allows any operation, effectively disabling type checking for that value.

**Q: What is the difference between `TypeAlias` and `NewType`?**
**A:** A type alias (e.g., `UserId = int`) creates an equivalent name for a type; the type checker treats them as identical. `NewType` (e.g., `UserId = NewType('UserId', int)`) creates a distinct subtype; the type checker will prevent you from passing a raw `int` where a `UserId` is expected, helping catch logic errors.

**Q: How do you handle circular imports when using type hints?**
**A:** Use the `TYPE_CHECKING` constant from the `typing` module to wrap imports that are only needed for type hints. These imports will only occur during static analysis and not at runtime. Additionally, use string literals (postponed evaluation) or `from __future__ import annotations` (Python 3.7+).

**Q: What is "Static Duck Typing" in Python?**
**A:** It refers to `typing.Protocol`. It allows you to define a set of methods that a type must have to be compatible, similar to interfaces in other languages, but without requiring the type to explicitly inherit from the Protocol.

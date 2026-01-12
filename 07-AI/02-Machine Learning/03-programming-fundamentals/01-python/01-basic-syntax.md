---
---

## Summary
Python is a high-level, interpreted language known for its readability ("executable pseudocode") and vast ecosystem, particularly in AI/ML. Unlike Go, it is dynamically typed and uses significant whitespace (indentation) instead of braces.

## Detailed Explanation

### Core Syntax
*   **Variables**: No type declaration. `x = 5`.
*   **Indentation**: Defines blocks. No `{ }`.
    ```python
    if x > 0:
        print("Positive")
    ```
*   **Functions**:
    ```python
    def add(a, b):
        return a + b
    ```
*   **Data Structures**: Lists `[1, 2]`, Dicts `{"key": "val"}`, Tuples `(1, 2)`.

### Object-Oriented
Everything is an object. Classes are first-class citizens.
```python
class Dog:
    def __init__(self, name):
        self.name = name
```

## Go-Specific Context/Examples

Comparison for a Go developer learning Python:

1.  **Typing**: Go is Static (`var x int`), Python is Dynamic (`x = 10` then `x = "hi"` is valid).
2.  **Error Handling**: Go uses `if err != nil`. Python uses `try/except`.
3.  **Concurrency**: Go has Goroutines/Channels. Python has `asyncio` (but limited by GIL - Global Interpreter Lock).

### List Comprehension vs For Loop
**Python**:
```python
squares = [x**2 for x in range(10)]
```
**Go**:
```go
squares := []int{}
for i := 0; i < 10; i++ {
    squares = append(squares, i*i)
}
```

## Interview Questions

**Q: What is the GIL?**
**A:** Global Interpreter Lock. A mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecodes at once. This limits CPU-bound parallelism in Python (unlike Go).

**Q: What are `*args` and `**kwargs`?**
**A:** They allow passing variable numbers of arguments. `*args` is a tuple of positional arguments. `**kwargs` is a dictionary of keyword arguments.

**Q: Why is Python preferred for AI/ML over Go?**
**A:** Not because of speed (Python is slow), but because of **C-bindings** and ecosystem. Libraries like PyTorch and NumPy are wrappers around highly optimized C/C++ code. Python provides the easy API surface to orchestrate these C kernels.

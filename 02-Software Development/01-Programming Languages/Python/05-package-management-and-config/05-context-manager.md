# Context Managers

## Summary
Context Managers manage resources (files, network connections, locks) to ensure they are properly acquired and released. The `with` statement is the syntactic sugar used to invoke them.

## Detailed Explanation

### The `with` Statement
It ensures cleanup code runs even if an error occurs inside the block.

```python
# Without Context Manager
f = open("file.txt", "w")
try:
    f.write("Hello")
finally:
    f.close() # Must manually close

# With Context Manager
with open("file.txt", "w") as f:
    f.write("Hello")
# f.close() is called automatically here
```

### How It Works: The Protocol
A class becomes a context manager if it implements:
1.  **`__enter__(self)`**: Sets up the resource. Returns the object to be bound to `as variable`.
2.  **`__exit__(self, exc_type, exc_val, exc_tb)`**: Cleans up. Handles exceptions if they occurred.

```python
class FileManager:
    def __init__(self, filename):
        self.filename = filename

    def __enter__(self):
        self.file = open(self.filename, 'w')
        return self.file

    def __exit__(self, exc_type, exc_value, traceback):
        self.file.close()

with FileManager("test.txt") as f:
    f.write("Testing")
```

### `contextlib`
The `contextlib` module allows creating context managers using generators and the `@contextmanager` decorator, avoiding the need for a full class.

```python
from contextlib import contextmanager

@contextmanager
def open_file(name):
    f = open(name, 'w')
    try:
        yield f
    finally:
        f.close()

with open_file("test.txt") as f:
    f.write("Generator based")
```

## Interview Questions

**Q: What happens if an exception occurs inside a `with` block?**
**A:** The `__exit__` method is immediately called. It receives the exception details. If `__exit__` returns `True`, the exception is suppressed. If it returns `False` (or None), the exception is re-raised after cleanup.

**Q: Give an example of a Context Manager besides opening files.**
**A:**
*   **Locks**: `with lock:` (automatically acquires/releases thread locks).
*   **Database Transactions**: `with db.transaction():` (commits if success, rollbacks if exception).
*   **Testing**: `with pytest.raises(ValueError):` (asserts that a block raises an exception).

**Q: What is the main benefit of using `with`?**
**A:** It guarantees resource cleanup (leak prevention) and simplifies code by removing verbose `try...finally` blocks.

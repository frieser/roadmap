#Python
---
---

## Summary

The `doctest` module is a part of the Python Standard Library that searches for pieces of text that look like interactive Python sessions, executes them, and verifies that they work exactly as shown. Its primary goal is to ensure that a module's documentation (specifically docstrings) stays accurate and up-to-date by turning examples into executable tests. This approach is often called **executable documentation**.

## Detailed Explanation

### Testing Examples in Docstrings

`doctest` identifies tests by looking for the primary prompt (`>>>`) and the secondary prompt (`...`) used in the interactive Python shell (REPL).

```python
def add(a, b):
    """
    Returns the sum of a and b.

    >>> add(2, 3)
    5
    >>> add('a', 'b')
    'ab'
    """
    return a + b
```

When run, `doctest` will execute `add(2, 3)` and check if the result is `5`. If it matches, the test passes.

### Interactive Shell Simulation

`doctest` mimics the behavior of the Python REPL. This means:
- **Output Matching**: It captures `stdout` and compares it string-for-string with the expected output in the docstring.
- **Exceptions**: To test for exceptions, you include the traceback, but usually only the header and the final error message are required.
- **Blank Lines**: Blank lines in the output must be represented by `<BLANKLINE>` if they occur within the output.
- **Multi-line Statements**: Uses `...` for continuation lines.

Example with Exception:
```python
def divide(a, b):
    """
    >>> divide(10, 0)
    Traceback (most recent call last):
        ...
    ZeroDivisionError: division by zero
    """
    return a / b
```

### Pros and Cons vs Proper Unit Tests (unittest/pytest)

| Feature | `doctest` | `unittest` / `pytest` |
| :--- | :--- | :--- |
| **Location** | Inside docstrings (literate testing). | In separate test files. |
| **Clarity** | Extremely clear as it serves as documentation. | Better for complex logic but less "readable" as docs. |
| **Maintainability** | Brittle; small changes in formatting (whitespace) can break tests. | Robust; uses assertions rather than string matching. |
| **Features** | Limited (no built-in mocking, parameterization). | Rich (fixtures, mocking, extensive assertions). |
| **Scope** | Best for simple examples and API documentation. | Best for complex logic, integration, and edge cases. |

### Running doctest

1. **Within the code**:
   ```python
   if __name__ == "__main__":
       import doctest
       doctest.testmod()
   ```

2. **Via Command Line**:
   ```bash
   python -m doctest my_module.py
   ```
   Add `-v` for verbose output.

## Interview Questions

1. **How does `doctest` find tests in a Python file?**
   It scans docstrings for the `>>>` prompt, which signals the start of an interactive example. It also looks for the `__test__` variable in a module if defined.

2. **How do you handle floating-point numbers in `doctest`?**
   Floating-point numbers can be tricky due to precision issues. You can use the `# doctest: +ELLIPSIS` directive or simply round the result in the example: `>>> round(my_func(), 2)`.

3. **What is the `...` (ellipsis) used for in `doctest`?**
   In the input, it represents a multi-line statement. In the output, if the `ELLIPSIS` option is enabled, it acts as a wildcard to match any substring.

4. **When should you avoid using `doctest`?**
   Avoid `doctest` for complex logic, tests that require extensive setup/teardown, mocking external dependencies, or when the output format is highly variable (like timestamps).

5. **Can `doctest` run tests in external text files?**
   Yes, using `doctest.testfile("README.md")`, which is useful for verifying examples in external documentation.

# Basic Syntax

## Summary
Python is designed for readability, using indentation to define blocks of code instead of braces. It is a high-level, interpreted language with dynamic typing.

## Detailed Explanation

### Indentation
Python uses whitespace (indentation) to define scope.
*   **Standard**: 4 spaces per indentation level.
*   **Rule**: Consistent indentation is mandatory. Mixing tabs and spaces causes errors.

```python
def greet():
    # This block is indented
    print("Hello, World!")
```

### Comments
*   **Single-line**: Starts with `#`.
*   **Multi-line**: Typically uses triple quotes `"""` or `'''` (technically multiline strings, but used as comments/docstrings).

```python
# This is a comment
x = 5  # Inline comment

"""
This is a multi-line comment
or docstring.
"""
```

### Input and Output
*   **Output**: `print()` function.
*   **Input**: `input()` function (always returns a string).

```python
name = input("Enter your name: ")
print("Hello,", name)
```

### Variables
Variables are created when assigned a value. No explicit declaration command is needed.

```python
x = 5
y = "John"
```

## Interview Questions

**Q: Why does Python use indentation?**
**A:** To enforce readability and structure. It eliminates the need for curly braces `{}` and makes the code visually represent its logical structure.

**Q: What is the difference between a comment and a docstring?**
**A:** A comment (`#`) is ignored by the interpreter. A docstring (`"""..."""`) is a string literal stored in the `__doc__` attribute of the function, class, or module, used for documentation.

**Q: Is Python compiled or interpreted?**
**A:** Python is technically both. Source code is compiled to bytecode (`.pyc`), which is then executed by the Python Virtual Machine (interpreter).

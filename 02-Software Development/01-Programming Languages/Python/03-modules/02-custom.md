# Custom Modules and Packages

## Summary
A Python module is simply a file ending in `.py`. A Package is a directory containing modules and (usually) an `__init__.py` file. Structuring code into modules promotes reusability and organization.

## Detailed Explanation

### Creating a Module
Any Python file can be a module.

**File: `my_math.py`**
```python
def add(a, b):
    return a + b

PI = 3.14159
```

**File: `main.py`**
```python
import my_math

print(my_math.add(2, 3))
print(my_math.PI)
```

### The `__name__` Guard
When a script is run directly, its `__name__` variable is set to `"__main__"`. When imported, it is set to the module's name.

```python
# utils.py
def setup():
    print("Setting up...")

if __name__ == "__main__":
    # This block runs ONLY if utils.py is executed directly
    # It does NOT run if utils.py is imported
    setup()
    print("Running tests...")
```

### Packages
A package is a directory structure.

```
my_app/
├── __init__.py
├── main.py
└── utils/
    ├── __init__.py
    ├── string_utils.py
    └── math_utils.py
```

To import from this structure:
```python
# main.py
from utils import string_utils
from utils.math_utils import calc_sum
```

### `__init__.py`
*   **Role**: Initializes the package.
*   **Content**: Can be empty, or can expose specific functions to simplify imports.
*   **Implicit Namespace Packages**: Since Python 3.3, `__init__.py` is technically optional for namespace packages, but it is still recommended for regular packages to define them clearly.

### Module Search Path (`sys.path`)
When importing, Python searches directories in this order:
1.  Current directory.
2.  `PYTHONPATH` environment variable.
3.  Standard library directories.
4.  Site-packages (installed 3rd party libraries).

## Interview Questions

**Q: What is the purpose of `if __name__ == "__main__":`?**
**A:** It checks if the script is being run directly by the user or imported as a module. Code inside this block will not execute if the file is imported, preventing side effects (like running tests or main loops) during import.

**Q: What is `__init__.py` used for?**
**A:** It marks a directory as a Python package, allowing imports from it. It can also execute initialization code for the package or define `__all__` to control what is exported when using `from package import *`.

**Q: How does Python find modules?**
**A:** It scans the list of directories in `sys.path`. You can inspect this list at runtime (`import sys; print(sys.path)`) or modify `PYTHONPATH` before running the script.

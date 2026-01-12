# Built-in Modules

## Summary
Python comes with a "batteries included" philosophy, meaning it has a rich Standard Library available without installing anything. Modules are files containing Python code that can be imported and used.

## Detailed Explanation

### Importing Modules
There are several ways to import modules:

1.  **Import entire module**:
    ```python
    import math
    print(math.sqrt(16))
    ```
2.  **Import specific items**:
    ```python
    from math import sqrt, pi
    print(sqrt(16))
    ```
3.  **Import with alias**:
    ```python
    import numpy as np
    import math as m
    ```
4.  **Import all (discouraged)**:
    ```python
    from math import * # Pollutes namespace, hard to track origins
    ```

### Essential Standard Libraries

#### 1. File System & OS (`os`, `sys`, `pathlib`)
*   **`os`**: Interact with the operating system (env vars, file paths).
*   **`sys`**: Interact with the Python interpreter (path, recursion limit, argv).
*   **`pathlib`**: Object-oriented filesystem paths (modern replacement for `os.path`).

```python
import sys
from pathlib import Path

print(sys.argv) # Command line arguments
path = Path("folder") / "file.txt" # Path manipulation
```

#### 2. Data & Math (`math`, `random`, `json`, `datetime`)
*   **`json`**: Parse and generate JSON.
*   **`datetime`**: Dates and times.
*   **`random`**: Random number generation.

```python
import json
data = {"name": "Alice", "age": 30}
json_str = json.dumps(data) # To string
parsed = json.loads(json_str) # To dict
```

#### 3. Data Structures (`collections`, `itertools`, `functools`)
*   **`collections`**: Specialized container datatypes (`deque`, `Counter`, `defaultdict`).
*   **`itertools`**: Iterator building blocks (`product`, `cycle`, `chain`).

```python
from collections import Counter
c = Counter(['a', 'b', 'c', 'a', 'b', 'b'])
print(c.most_common(1)) # [('b', 3)]
```

## Interview Questions

**Q: What is the difference between `import module` and `from module import item`?**
**A:** `import module` loads the module and creates a reference to it in the current namespace. You access items via `module.item`. `from module import item` loads the module but only imports the specific item into the current namespace, allowing you to use `item` directly.

**Q: Why is `from module import *` discouraged?**
**A:** It pollutes the global namespace, making it unclear where functions or classes came from. It can also overwrite existing variables with the same name.

**Q: Which module is modern: `os.path` or `pathlib`?**
**A:** `pathlib` (introduced in Python 3.4) is the modern, object-oriented approach and is generally preferred over string-based `os.path` manipulation.

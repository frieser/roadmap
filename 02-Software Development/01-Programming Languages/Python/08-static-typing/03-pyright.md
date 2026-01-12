#Python
---
---

## Summary
**Pyright** is a high-performance static type checker for Python, developed by Microsoft. Unlike `mypy`, which is written in Python, Pyright is written in **TypeScript** and runs on **Node.js**. It is designed for speed and scalability, making it ideal for large-scale Python codebases. It is the core engine behind **Pylance**, the default language server for Python in Visual Studio Code.

## Detailed Explanation

### Performance vs mypy
Pyright is significantly faster than mypy, often by a factor of 3x to 5x. This is achieved through:
- **Language**: Being written in TypeScript allows it to leverage the highly optimized V8 engine and multi-threading capabilities of Node.js.
- **Lazy Evaluation**: It only analyzes the files needed for the current context (e.g., the file currently open in the editor) rather than checking the entire project upfront unless explicitly requested.
- **Persistence**: When used as a language server, it maintains a persistent state in memory, allowing for nearly instantaneous incremental checks.

### VS Code Integration (Pylance)
Pyright's most common usage is through **Pylance**, Microsoft's "pro" language server for Python in VS Code.
- Pylance wraps the Pyright engine and adds additional features like semantic highlighting, auto-imports, and better IntelliSense.
- It provides a "Strict Mode" toggle in settings that enables the most rigorous type checking rules.

### Configuration
Pyright can be configured via `pyrightconfig.json` or within `pyproject.toml`.

#### Option 1: `pyproject.toml`
```toml
[tool.pyright]
include = ["src"]
exclude = ["**/node_modules", "**/__pycache__"]
venvPath = "."
venv = ".venv"
pythonVersion = "3.10"
typeCheckingMode = "strict"
```

#### Option 2: `pyrightconfig.json`
```json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__"],
  "strict": ["src/core"],
  "reportMissingImports": true,
  "pythonVersion": "3.10"
}
```

### Type Inference Capabilities
Pyright features more aggressive and accurate type inference than mypy in several areas:
- **Closures**: Better at tracking types through nested functions.
- **Recursive Types**: Excellent support for complex recursive structures (e.g., JSON-like structures).
- **Generic Classes**: More robust handling of generic class inheritance and variance.

```python
# Example of complex inference
from typing import TypedDict, List, Union

class User(TypedDict):
    id: int
    name: str

# Pyright accurately infers types in complex list comprehensions
users: List[User] = [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
names = [u["name"] for u in users] # Inferred as List[str]
```

## Interview Questions
1. **How does Pyright's architecture differ from Mypy's, and why does it matter?**
   - Pyright is written in TypeScript/Node.js, while Mypy is written in Python. This allows Pyright to utilize Node.js's multi-threading and V8 optimizations, leading to significantly faster performance on large projects.
2. **What is the relationship between Pyright and Pylance?**
   - Pyright is the open-source type-checking engine, while Pylance is Microsoft's closed-source language server for VS Code that uses Pyright as its core engine but adds proprietary features like semantic highlighting and better IntelliSense.
3. **How do you configure a specific virtual environment for Pyright?**
   - You can use the `venvPath` (path to the folder containing venvs) and `venv` (name of the specific venv folder) settings in `pyrightconfig.json` or `pyproject.toml`.
4. **What are the different type-checking modes in Pyright?**
   - Pyright offers `off`, `basic`, and `strict`. `strict` mode enables all type-checking rules and is recommended for high-quality production codebases.

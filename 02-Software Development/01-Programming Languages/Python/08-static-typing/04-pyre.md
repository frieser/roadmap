#Python
---
---

## Summary
Pyre is a high-performance static type checker for Python developed by Meta. Written in OCaml, it is designed for speed and scalability, making it ideal for large-scale codebases. Beyond standard type checking, it includes **Pysa** (Python Static Analyzer), a powerful security tool that uses taint analysis to identify potential vulnerabilities like remote code execution or SQL injection by tracking data flow from sources to sinks.

## Detailed Explanation

### 1. Speed and Incremental Checks
Pyre's architecture is optimized for developer productivity in massive repositories. It achieves this through several key mechanisms:
- **OCaml Backend**: Unlike mypy (written in Python/C), Pyre's core is implemented in OCaml, allowing for highly efficient parallel processing and memory management.
- **Persistent Daemon**: Pyre runs a background server (daemon) that maintains a representation of the code's type environment in memory.
- **Watchman Integration**: It uses Meta's `watchman` tool to monitor file changes. When a file is saved, Pyre only re-checks the affected parts of the dependency graph, providing near-instant feedback.

```bash
# Installing Pyre and Watchman
pip install pyre-check
# Note: watchman is usually installed via system package manager (e.g., brew install watchman)

# Initialize Pyre in a project
pyre init

# Start checking
pyre
```

### 2. Pysa (Python Static Analyzer)
Pysa is a security-focused tool bundled with Pyre. It performs **Taint Analysis**, which tracks the flow of "tainted" (potentially malicious or sensitive) data through an application.

- **Sources**: Where tainted data enters (e.g., `request.GET`, `input()`).
- **Sinks**: Where tainted data should not go without sanitization (e.g., `eval()`, `os.system()`, `cursor.execute()`).
- **Sanitizers**: Functions that clean the data, removing the "taint."
- **Rules**: Definitions that specify which source-to-sink flows constitute a vulnerability.

```python
# Example of a flow Pysa would catch
def handle_request(request):
    # 'user_id' is a Source (from HTTP request)
    user_id = request.GET.get("id")
    
    # Passing user_id directly to a Sink (SQL execution) is a vulnerability
    # Pysa identifies this flow and flags it.
    query = f"SELECT * FROM users WHERE id = {user_id}"
    db.execute(query) 
```

### 3. Pyre Query
One unique feature is the ability to query the type environment directly. This is useful for building custom tooling or debugging complex type issues.

```bash
# Query the type of an expression in a file
pyre query "type(test.py:5:10)"

# Find all definitions of a symbol
pyre query "definition(my_module.MyClass)"
```

### 4. Differences from Mypy
While both tools aim to improve Python code quality through static typing, they have distinct philosophies:

| Feature | Mypy | Pyre |
| --- | --- | --- |
| **Language** | Python (with some C extensions) | OCaml |
| **Performance** | Good for small/medium projects | Optimized for multi-million line repos |
| **Security** | Minimal (basic type safety) | Integrated security analysis (Pysa) |
| **Incremental** | Uses filesystem cache | Uses persistent daemon + Watchman |
| **Inference** | Strong, but can be slow | Focuses on fast local inference |
| **Environment** | Standard in the ecosystem | Heavily used at Meta/Instagram |

## Interview Questions

**Q: What is the primary difference between Pyre and Pysa?**
**A:** Pyre is the general-purpose static type checker used to find type errors (e.g., passing a string to a function expecting an int). Pysa is a security-focused tool built on top of Pyre that uses taint analysis to find data flow vulnerabilities (e.g., user input reaching a database query).

**Q: How does Pyre maintain speed in very large repositories?**
**A:** Pyre uses an OCaml-based backend for high performance, runs a persistent daemon to keep the type environment in memory, and integrates with `watchman` for efficient incremental checking of only modified files.

**Q: Explain the concept of "Taint Analysis" in Pysa.**
**A:** Taint analysis is a form of information flow analysis. It marks data from untrusted "sources" (like HTTP requests) as tainted and tracks it as it moves through the program. If tainted data reaches a "sink" (a sensitive function like `eval`) without being "sanitized," Pysa reports a security vulnerability.

**Q: Why would a developer use 'pyre query'?**
**A:** Developers use `pyre query` to programmatically extract information from the type checker, such as the type of a specific variable, the definition location of a class, or the list of all attributes of a module. This is powerful for building IDE plugins or automated refactoring scripts.

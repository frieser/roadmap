#Python
---
---

## Summary
Mypy is a static type checker for Python that leverages type hints (introduced in PEP 484) to identify potential bugs and inconsistencies without executing the code. It combines the benefits of dynamic typing (Python's flexibility) with the robustness of static typing, allowing for gradual adoption in existing codebases. By verifying type annotations, mypy improves code readability, catches refactoring errors early, and enhances IDE support.

## Detailed Explanation

### How it works
Mypy performs a static analysis of Python source code. It reads the code and any associated type hint annotations, then checks if the usage of variables, functions, and classes matches these hints. It does not affect runtime performance because type hints are ignored by the Python interpreter. Mypy uses a sophisticated type inference system to determine types where explicit annotations are missing, although it works best when critical boundaries (like function signatures) are explicitly typed.

### Configuration
Mypy can be configured using `mypy.ini`, `.mypy.ini`, `setup.cfg`, or `pyproject.toml`.

#### **pyproject.toml (Modern approach)**
```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_unreachable = true
pretty = true
show_error_codes = true

[[tool.mypy.overrides]]
module = "external_lib.*"
ignore_missing_imports = true
```

#### **mypy.ini (Legacy/Standard approach)**
```ini
[mypy]
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
show_error_codes = True

[mypy-external_lib.*]
ignore_missing_imports = True
```

### Strict Mode
Using the `--strict` flag (or `strict = true` in config) enables all strictness-related flags. This is highly recommended for new projects. It requires all functions to have type annotations and disallows many common "unsafe" patterns.

It is equivalent to enabling:
- `--warn-unused-configs`
- `--disallow-any-generics`
- `--disallow-subclassing-any`
- `--disallow-untyped-calls`
- `--disallow-untyped-defs`
- `--disallow-incomplete-defs`
- `--check-untyped-defs`
- `--disallow-untyped-decorators`
- `--no-implicit-optional`
- `--warn-redundant-casts`
- `--warn-unused-ignores`
- `--warn-return-any`
- `--no-implicit-reexport`
- `--strict-equality`

### Ignoring Errors
You can suppress mypy errors using `# type: ignore` comments. It is best practice to specify the error code to avoid hiding unrelated bugs.

```python
from typing import Any

def process_data(data: Any) -> int:
    # Suppress a specific attribute error
    return data.value  # type: ignore[attr-defined]

# Suppress all errors on a line (not recommended)
x = "string" + 123  # type: ignore
```

In configuration, you can use `ignore_errors = True` for specific modules:
```ini
[mypy-legacy_module.*]
ignore_errors = True
```

### Common Flags
- `--strict`: Enable all strict mode flags.
- `--ignore-missing-imports`: Do not report errors if an imported module cannot be found (useful for libs without stubs).
- `--check-untyped-defs`: Type check the body of functions without type annotations.
- `--show-error-codes`: Display error codes in the output (e.g., `[attr-defined]`).
- `--pretty`: Use colors and visual markers for errors.
- `--python-version X.Y`: Check compatibility with a specific Python version.
- `--install-types`: Non-interactively install missing type stubs (usually with `--non-interactive`).

## Interview Questions

**Q: What is the difference between static type checking with mypy and runtime type checking?**
**A:** Static type checking (mypy) happens before the code runs, analyzing source code to find potential type mismatches with zero runtime overhead. Runtime type checking (like using `isinstance()`) happens during execution and can catch errors that depend on dynamic values, but it adds performance overhead.

**Q: What does the `--strict` flag in mypy do?**
**A:** The `--strict` flag is a shortcut that enables a collection of strictness-related flags. It enforces comprehensive type hints (no untyped functions), disallows `Any` in many contexts, and enables various warnings for redundant or unreachable code.

**Q: How do you handle a third-party library that doesn't provide type hints or stubs?**
**A:** You can use the `--ignore-missing-imports` flag or add a module-specific override in your config file (`ignore_missing_imports = True`). Alternatively, you can create a `.pyi` stub file or install community-maintained stubs from `typeshed` (e.g., `pip install types-requests`).

**Q: What is a `.pyi` file?**
**A:** A `.pyi` file is a "stub file" that contains only type information for a module, without the implementation. Mypy uses these files to understand types for libraries written in C or those lacking inline type hints.

**Q: Why is it recommended to use `# type: ignore[error-code]` instead of just `# type: ignore`?**
**A:** Specifying the error code ensures you only suppress the specific issue you intend to ignore. It prevents accidentally hiding new or unrelated type errors that might be introduced on the same line in the future.

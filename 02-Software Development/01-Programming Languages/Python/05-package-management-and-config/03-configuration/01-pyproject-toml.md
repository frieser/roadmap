# pyproject.toml (PEP 621)

## Summary
`pyproject.toml` is the new standard configuration file for Python projects. It unifies tool configuration (black, ruff, pytest) and project metadata (name, version, dependencies) into a single file, replacing `setup.py`, `requirements.txt`, and `setup.cfg`.

## Detailed Explanation

### Key Sections

#### 1. `[build-system]`
Defines how to build the package (e.g., using `setuptools`, `hatchling`, or `poetry-core`).

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

#### 2. `[project]` (PEP 621 Standard)
Standardized metadata.

```toml
[project]
name = "my-awesome-app"
version = "0.1.0"
description = "A great app"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28",
    "fastapi"
]
```

#### 3. `[tool.*]`
Configuration for various tools.

```toml
[tool.ruff]
line-length = 88

[tool.pytest.ini_options]
addopts = "-ra -q"
```

### Benefits
*   **Standardization**: Works across different build backends (Flit, Poetry, Hatch).
*   **Centralization**: One file for everything.
*   **Declarative**: Easy to read and parse (TOML format).

## Interview Questions

**Q: What is PEP 621?**
**A:** It is the Python Enhancement Proposal that standardized how project metadata (dependencies, version, author) is written in `pyproject.toml`. Before this, every tool (Poetry, Flit) used its own custom schema.

**Q: Can I delete `setup.py` now?**
**A:** In most cases, yes. `pyproject.toml` replaces the need for `setup.py`. However, for complex C-extensions or dynamic build logic, a `setup.py` might still be required (though often it just reads from toml).

**Q: How do I specify optional dependencies (extras)?**
**A:** In the `[project.optional-dependencies]` section.
```toml
[project.optional-dependencies]
dev = ["pytest", "black"]
```
Installed via `pip install .[dev]`.

# Pip

## Summary
`pip` (Pip Installs Packages) is the standard package manager for Python. It installs packages from PyPI and manages dependencies.

## Detailed Explanation

### Basic Usage
*   **Install**: `pip install requests`
*   **Uninstall**: `pip uninstall requests`
*   **List**: `pip list` (shows installed packages)
*   **Freeze**: `pip freeze > requirements.txt` (outputs installed packages and versions to a file).

### Requirements Files
Standard way to track dependencies in legacy/simple projects.
**requirements.txt**:
```text
requests==2.28.1
numpy>=1.21.0
```
Install all: `pip install -r requirements.txt`

### Limitations (The "Dependency Hell")
*   **No Dependency Resolution (Historically)**: Older versions of pip simply installed what was asked. If Package A needed `Lib==1.0` and Package B needed `Lib==2.0`, pip might install conflicting versions. (Modern `pip` has a resolver, but it's slower/less robust than Poetry/uv).
*   **No Lock File**: `requirements.txt` is not a true lock file unless you pin *every* transitive dependency manually.
*   **Global Installs**: By default, pip installs globally or to the user directory. It doesn't automatically manage virtual environments.

## Interview Questions

**Q: What is the purpose of `pip freeze`?**
**A:** It lists all installed packages and their specific versions in a format compatible with `requirements.txt`. It is used to capture the current environment state.

**Q: Why should you avoid `pip install` without a virtual environment?**
**A:** It installs packages into the global Python environment. This causes conflicts when different projects on the same machine require different versions of the same library (Dependency Hell).

**Q: How does `pip install -e .` work?**
**A:** It installs the current package in "editable" mode. Changes to the source code are immediately reflected in the environment without needing to reinstall. Useful for development.

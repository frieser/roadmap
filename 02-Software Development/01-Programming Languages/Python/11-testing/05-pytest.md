#Python
---
---

## Summary
**pytest** is a mature, full-featured Python testing tool that helps you write better programs. It is widely considered the industry standard for Python testing due to its simple syntax, powerful features, and extensive plugin ecosystem. Unlike the built-in `unittest` module, which requires boilerplate classes and specific assertion methods, `pytest` allows writing tests as simple functions and uses standard Python `assert` statements, making tests more readable and maintainable.

## Detailed Explanation

### 1. Fixtures
Fixtures are functions that provide a defined, reliable, and consistent context for tests. They are the cornerstone of `pytest`'s dependency injection system.
- **Dependency Injection**: Test functions "request" fixtures by including the fixture name as an argument.
- **Scopes**: Fixtures can be scoped to control how often they are invoked:
    - `function` (default): Run once per test function.
    - `class`: Run once per test class.
    - `module`: Run once per module (.py file).
    - `package`: Run once per package.
    - `session`: Run once per test session.
- **Teardown**: Uses the `yield` keyword instead of `return`. Code after `yield` executes after the test completes.

```python
import pytest

@pytest.fixture(scope="module")
def db_connection():
    # Setup: Create connection
    conn = {"status": "connected"}
    yield conn
    # Teardown: Close connection
    conn["status"] = "closed"

def test_db_query(db_connection):
    assert db_connection["status"] == "connected"
```

### 2. Parametrization
Parametrization allows running the same test function multiple times with different input values, reducing code duplication.
- Use `@pytest.mark.parametrize` to define multiple sets of arguments.

```python
import pytest

@pytest.mark.parametrize("a, b, expected", [
    (1, 1, 2),
    (2, 3, 5),
    (10, 5, 15)
])
def test_addition(a, b, expected):
    assert a + b == expected
```

### 3. Plugins
Pytest features a robust plugin architecture with over 800+ community-maintained plugins.
- **pytest-cov**: Produces coverage reports.
- **pytest-xdist**: Runs tests in parallel across multiple CPUs.
- **pytest-django / pytest-flask**: Specialized integrations for web frameworks.
- **pytest-mock**: Thin-wrapper around the `unittest.mock` library.

### 4. conftest.py
`conftest.py` is a special file used to share fixtures, hooks, and configurations across multiple test files within a directory or subdirectories.
- Pytest automatically discovers `conftest.py` files.
- Fixtures defined in `conftest.py` can be used by any test in that package without needing to import them.

### 5. Markers
Markers are used to categorize tests or modify their execution behavior.
- **Built-in Markers**:
    - `@pytest.mark.skip(reason=...)`: Unconditionally skip a test.
    - `@pytest.mark.skipif(condition, ...)`: Skip if a condition is met.
    - `@pytest.mark.xfail`: Mark a test as expected to fail.
- **Custom Markers**: Categorize tests (e.g., `@pytest.mark.slow`) and run them specifically using `pytest -m slow`.

### 6. Assert Introspection
One of `pytest`'s most powerful features is its ability to provide detailed information about why an `assert` statement failed without requiring specialized `assert*` methods.
- When an assertion fails, `pytest` reinterprets the statement and provides a detailed "diff" of the values involved.

```python
# Example output for a failing list comparison:
# E       AssertionError: assert [1, 2, 3] == [1, 2, 4]
# E         At index 2 diff: 3 != 4
# E         Use -v to get more diff
```

## Interview Questions

1. **How does pytest's fixture system differ from unittest's setUp/tearDown?**
   - *Answer*: `pytest` fixtures are modular and use dependency injection. They can have different scopes (session, module, etc.) and handle teardown via `yield`, making them more flexible than the class-based `setUp`/`tearDown` in `unittest`.

2. **What is the purpose of the `conftest.py` file?**
   - *Answer*: It is used to share fixtures and configuration across multiple test files in a directory tree without requiring explicit imports.

3. **How do you run tests in parallel using pytest?**
   - *Answer*: By using the `pytest-xdist` plugin and running with the `-n` flag (e.g., `pytest -n auto`).

4. **What are the different scopes available for a pytest fixture?**
   - *Answer*: `function`, `class`, `module`, `package`, and `session`.

5. **How can you mark a test to be skipped if the Python version is less than 3.10?**
   - *Answer*: Using `@pytest.mark.skipif(sys.version_info < (3, 10), reason="requires python3.10")`.

6. **Explain 'Assert Introspection' in pytest.**
   - *Answer*: Pytest intercepts standard Python `assert` calls and provides detailed failure messages by showing the values of the variables and expressions involved in the failure, eliminating the need for `self.assertEqual`-style methods.

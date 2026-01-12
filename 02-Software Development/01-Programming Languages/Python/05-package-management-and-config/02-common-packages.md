# Common Packages

## Summary
While the standard library is vast, the Python ecosystem relies heavily on third-party packages. Knowing the "standard" third-party libraries is crucial for professional development.

## Essential Libraries

### 1. Web & API
*   **`requests`**: "HTTP for Humans". The standard for making HTTP requests. Much simpler than built-in `urllib`.
*   **`fastapi`**: Modern, fast web framework for building APIs with Python 3.6+ based on standard Python type hints.
*   **`flask`**: Lightweight WSGI web application framework.

### 2. Testing
*   **`pytest`**: The de-facto testing framework. Features simple syntax (no class boilerplate), powerful fixtures, and rich ecosystem of plugins.
*   **`coverage`**: Measures code coverage of tests.

### 3. Data & Science
*   **`numpy`**: Fundamental package for scientific computing (arrays, matrices).
*   **`pandas`**: Data structures (DataFrames) and analysis tools.
*   **`pydantic`**: Data validation using Python type hints. Used heavily by FastAPI.

### 4. Tooling & Quality
*   **`ruff`**: An extremely fast Python linter and formatter, written in Rust. Replaces Flake8, Black, and isort.
*   **`black`**: The uncompromising code formatter.
*   **`mypy`**: Static type checker.

### 5. Utilities
*   **`tqdm`**: Instantly make loops show a smart progress meter.
*   **`python-dotenv`**: Reads key-value pairs from a `.env` file and adds them to environment variables.

## Interview Questions

**Q: Why use `requests` instead of `urllib`?**
**A:** `requests` provides a much more user-friendly API. It handles connection pooling, sessions, encoding, and JSON content automatically, whereas `urllib` requires verbose boilerplate code.

**Q: What is the main advantage of `pydantic`?**
**A:** It enforces type hints at runtime. If you define a model `class User(BaseModel): id: int`, Pydantic ensures `id` is an int (or casts it if possible), providing robust data validation and serialization.

**Q: Why is `pytest` preferred over `unittest`?**
**A:** `pytest` uses simple `assert` statements instead of verbose `self.assertEqual` methods. It also supports "fixtures" which are a more powerful and modular way to handle setup/teardown logic than `setUp/tearDown` methods.

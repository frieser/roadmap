# Poetry

## Summary
Poetry is a dependency management and packaging tool. Before `uv` arrived, it was the gold standard for modern Python development. It handles dependency resolution, lock files (`poetry.lock`), and publishing to PyPI.

## Detailed Explanation

### The Workflow
1.  **Init**: `poetry init` (Interactive setup of `pyproject.toml`).
2.  **Add**: `poetry add flask` (Resolves dependency, installs it, updates toml and lock file).
3.  **Install**: `poetry install` (Installs from lock file, creates virtualenv if missing).
4.  **Shell**: `poetry shell` (Activates the virtualenv).

### Dependency Resolution
Poetry uses a sophisticated resolver (PubGrub) to find a set of package versions that satisfy all constraints. If a conflict exists (e.g., Lib A wants `requests<2` and Lib B wants `requests>2`), Poetry will fail explicitly with an explanation, preventing broken environments.

### Publishing
Poetry simplifies publishing libraries.
```bash
poetry build  # Creates .tar.gz and .whl
poetry publish # Uploads to PyPI (requires auth)
```

### pyproject.toml Integration
Poetry championed the use of `pyproject.toml` for dependencies long before it became the official standard (PEP 621). Note that older Poetry projects use a `[tool.poetry]` section, while newer ones migrate to standard `[project]`.

## Interview Questions

**Q: What is the purpose of `poetry.lock`?**
**A:** It freezes the exact versions of all dependencies (and transitive dependencies) installed. This ensures that every developer and the CI/CD server use the *exact same* environment, preventing "it works on my machine" bugs.

**Q: How does Poetry handle virtual environments?**
**A:** By default, it creates a virtual environment in a specialized cache directory (e.g., `~/.cache/pypoetry/virtualenvs`). You can configure it to create `.venv` in the project root: `poetry config virtualenvs.in-project true`.

**Q: Why choose Poetry over Pip?**
**A:** For the dependency resolver and lock file. Pip (historically) didn't guarantee deterministic installs. Poetry guarantees that if `poetry install` succeeds, the environment is consistent.

# pre-commit Hook

## Summary
The `pre-commit` hook is the most popular and versatile client-side hook. it runs before you even type a commit message.

## Detailed Explanation
- **When it runs:** Immediately after running `git commit`.
- **Primary Use:** Static analysis, linting, formatting, and sanity checks.

### AI Engineering Use Case
- **Linting:** Run `pylint`, `flake8`.
- **Formatting:** Run `black`, `isort`.
- **Security:** Run `detect-secrets`.
- **Notebooks:** Run `nbstripout` or `nb-clean`.
- **Configs:** Validate `config.yaml` against a Pydantic schema.

## Interview Questions
1.  **How do you bypass the `pre-commit` hook?**
    Using the `--no-verify` flag: `git commit --no-verify -m "message"`.
2.  **What is the `pre-commit` framework?**
    A Python-based multi-language package manager for Git hooks. It simplifies managing and sharing hooks across a team.

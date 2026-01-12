# Branch Naming

## Summary
Consistent branch naming conventions make it easy to identify the purpose of a branch and manage the repository. Kebab-case is the standard.

## Detailed Explanation

### Common Prefixes
*   **`feature/`** or **`feat/`**: New capabilities (e.g., `feat/login-page`).
*   **`bugfix/`** or **`fix/`**: Bug repairs (e.g., `fix/header-alignment`).
*   **`hotfix/`**: Critical production fixes (usually branched from `main` or `release`).
*   **`chore/`**: Maintenance tasks (e.g., `chore/update-deps`).
*   **`docs/`**: Documentation only changes.
*   **`refactor/`**: Code restructuring without behavior change.

### Naming Rules
*   Use **kebab-case** (lowercase with hyphens): `feature/new-login-flow`.
*   Avoid special characters.
*   Include issue number if applicable: `feat/issue-123-add-login`.

### Go-specific Context
In Go modules, if you are working on a major version upgrade, you might see branches like `v2-dev`.

## Interview Questions
**Q: Can you use uppercase in branch names?**
**A:** You *can*, but it is discouraged because some file systems (Windows/macOS) are case-insensitive, which can cause Git confusion. Stick to lowercase.

**Q: What character acts as a "folder" separator in branch names?**
**A:** The forward slash `/`. Git GUIs often group branches by prefix (e.g., all `feature/*` branches in a folder).

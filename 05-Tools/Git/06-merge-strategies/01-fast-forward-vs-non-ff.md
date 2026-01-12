# Fast-Forward vs Non-Fast-Forward

## Summary
When merging, Git defaults to a "Fast-Forward" (FF) strategy if the target branch has not diverged. This simply moves the pointer. A "Non-Fast-Forward" (No-FF) merge forces the creation of a merge commit, preserving the existence of the feature branch in history.

## Detailed Explanation

### Fast-Forward (Default)
*   **Condition**: The `main` branch has not advanced since you created `feature`.
*   **Result**: Linear history. It looks like you developed directly on `main`.
*   **Command**: `git merge feature`

### Non-Fast-Forward (--no-ff)
*   **Condition**: Forced by flag or if `main` has diverged.
*   **Result**: Creates a "Merge Commit". Keeps the history of the `feature` branch grouped together.
*   **Command**: `git merge --no-ff feature`

### Go-specific Context
In Go open-source projects, maintainers often prefer **Squash Merges** (which is a form of FF-like linearity but with one commit) or **Rebase Merges** to keep a clean, linear history, avoiding the "railroad tracks" look of frequent merge commits.

## Interview Questions
**Q: Why might you want to force a merge commit (`--no-ff`)?**
**A:** To preserve the historical context that a set of commits belonged to a specific feature branch. If you FF, you lose the information that those commits were developed together.

**Q: How do you configure Git to always create a merge commit?**
**A:** `git config --global merge.ff false` (Not recommended for most workflows today).

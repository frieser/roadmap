# Clean Git History

## Summary
A clean Git history is linear, easy to understand, and free of "noise". It allows new developers to understand *how* the project evolved and makes debugging (using `git bisect`) possible.

## Detailed Explanation

### Characteristics of Clean History
1.  **Atomic Commits**: Each commit does one thing and passes tests.
2.  **Meaningful Messages**: No "fix", "wip", "update" messages.
3.  **Linearity**: Avoid unnecessary merge commits from `git pull` (prefer rebase).
4.  **No Binary Bloat**: Large binaries should not be in git (use Git LFS).

### How to achieve it
*   **Squash** your WIP commits before merging.
*   **Rebase** your feature branch on main to keep it up to date, rather than merging main into it constantly.

### Go-specific Context
In Go, keeping `go.mod` and `go.sum` updates in separate commits from code logic can sometimes make the history clearer, especially for large dependency upgrades.

## Interview Questions
**Q: What is "commit noise"?**
**A:** Commits like "fix typo", "formatting", "forgot file" that clutter the history without adding semantic value.

**Q: How do you remove a large file that was accidentally committed?**
**A:** `git filter-branch` or the faster `git-filter-repo` tool. Simply deleting it with `rm` and committing only removes it from the *current* snapshot, not the history.

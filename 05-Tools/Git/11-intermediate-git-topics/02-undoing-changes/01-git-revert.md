# Git Revert

## Summary
`git revert` is a safe way to undo a commit. Instead of deleting the commit from history (like `reset`), it creates a **new** commit that introduces the *inverse* changes.

## Detailed Explanation

### Usage
```bash
git revert <commit-sha>
```
If commit A added a line, `git revert A` creates commit B that deletes that line.

### Why use it?
*   **Safety**: It preserves history.
*   **Collaboration**: Safe to use on public branches (`main`). If you used `git reset` on `main`, you would break history for everyone else.

### Go-specific Context
If a merged PR introduced a bug in a Go service, you should `git revert` the merge commit. This records that the feature was deployed and then rolled back, providing a clear audit trail.

## Interview Questions
**Q: Does `git revert` change the project history?**
**A:** It adds to the history; it does not modify past history.

**Q: Can you revert a revert?**
**A:** Yes. `git revert <revert-commit-sha>` will re-apply the original changes.

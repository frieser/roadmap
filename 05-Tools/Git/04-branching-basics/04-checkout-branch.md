# Checkout Branch

## Summary
Checking out a branch updates the files in the working directory to match the version stored in that branch, and tells Git to record new commits on that branch.

## Detailed Explanation

### Commands
*   **Old way**: `git checkout <branch-name>`.
*   **New way (Git 2.23+)**: `git switch <branch-name>`.

### Detached HEAD
If you check out a specific commit hash (instead of a branch name), you enter "Detached HEAD" state.
```bash
git checkout a1b2c3d
```
In this state, you can look around and make experimental changes, but if you commit, those commits belong to no branch and can be easily lost if you switch away.

### Go-specific Context
When debugging a regression in a Go app, you often checkout older commits to find where the bug was introduced.
```bash
git checkout v1.0.0
go test ./...
```

## Interview Questions
**Q: What is the difference between `git switch` and `git checkout`?**
**A:** `git checkout` is a Swiss Army knife that does many things (restore files, switch branches). `git switch` was introduced to be a dedicated, safer command just for switching branches.

**Q: How do you create a new branch and switch to it in one command?**
**A:** `git checkout -b <name>` or `git switch -c <name>`.

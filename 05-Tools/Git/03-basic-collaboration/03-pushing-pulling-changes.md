# Pushing and Pulling Changes

## Summary
Pushing and pulling are the primary mechanisms for synchronizing your local repository with a remote one. `git push` uploads your commits, while `git pull` downloads and merges commits from the remote.

## Detailed Explanation

### Pushing
Sends your committed changes to the remote.
```bash
git push <remote> <branch>
# Example
git push origin main
```
*   **-u (upstream)**: `git push -u origin main` links your local branch to the remote branch, so you can just type `git push` in the future.

### Pulling
Updates your current branch with changes from the remote.
```bash
git pull origin main
```
*   **Under the hood**: `git pull` = `git fetch` + `git merge`.

### Go-specific Context
If you are working on a Go module and someone else updates `go.mod` (adds a dependency), you must pull those changes.
After pulling, always run:
```bash
go mod tidy
```
This ensures your local `go.sum` and module cache are in sync with the new `go.mod`.

## Interview Questions
**Q: What happens if you try to push but your local history is behind the remote?**
**A:** Git will reject the push. You must first `git pull` (fetch and merge) the latest changes, resolve any conflicts, and then push.

**Q: What is `git push --force`?**
**A:** It overwrites the remote history with your local history. This is dangerous and should be avoided on shared branches as it can delete other people's work.

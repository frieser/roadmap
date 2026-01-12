# Git Worktree

## Summary
`git worktree` allows you to check out multiple branches of the same repository into different directories simultaneously. This means you can work on `feature-A` and `hotfix-B` at the same time without constantly switching branches and rebuilding.

## Detailed Explanation

### Usage
```bash
git worktree add ../my-repo-hotfix hotfix-branch
```
This creates a new folder `../my-repo-hotfix` linked to your main repo but checked out to a different branch.

### Use Cases
*   Running long tests on one branch while coding on another.
*   Comparing running versions of the app side-by-side.

### Go-specific Context
Since Go builds are fast, switching branches isn't too painful, but `git worktree` is excellent when you have local uncommitted changes you don't want to stash, or if you are running a long integration test suite (`go test -tags=integration ./...`) on one version while fixing a bug in another.

## Interview Questions
**Q: Do worktrees share the `.git` directory?**
**A:** Yes, they share the same object database and config, keeping disk usage low compared to cloning the repo twice.

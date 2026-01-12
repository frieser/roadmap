# Creating a Branch

## Summary
Branching allows you to diverge from the main line of development and continue to do work without messing with that main line. Creating a branch in Git is incredibly lightweight and fast.

## Detailed Explanation

### Commands
1.  **Create only**: `git branch <branch-name>` (Does not switch to it).
2.  **Create and Switch**: `git checkout -b <branch-name>` or `git switch -c <branch-name>`.

### Best Practices
*   Name branches descriptively: `feat/login-page`, `bug/fix-header`, `chore/update-deps`.
*   Always branch off `main` (or `develop`) unless you have a specific reason not to.

### Go-specific Context
When working on a Go feature:
```bash
# Good workflow
git checkout main
git pull origin main
git checkout -b feature/add-middleware
```
This ensures your new Go code is based on the latest stable version.

## Interview Questions
**Q: What is a branch in Git technically?**
**A:** A branch is simply a lightweight movable pointer to one specific commit.

**Q: How do you list all local branches?**
**A:** `git branch`. To see remote branches too, use `git branch -a`.

# Detached HEAD

## Summary
A "Detached HEAD" state occurs when HEAD points directly to a commit SHA instead of pointing to a branch name. You are "detached" from any branch.

## Detailed Explanation

### How to get there
```bash
git checkout <commit-sha>
# or
git checkout v1.0.0 (a tag)
```

### Implications
If you make commits in this state, they will **not** belong to any branch. If you switch away to another branch (`git checkout main`), your new commits will be orphaned and eventually garbage collected (lost).

### How to fix
If you made commits in detached HEAD that you want to keep:
```bash
git checkout -b new-branch-name
```
This creates a new branch pointing to your current commit, saving your work.

## Interview Questions
**Q: Is Detached HEAD an error state?**
**A:** No, it's a valid state for inspecting old code. It's only dangerous if you start developing new features without creating a branch.

**Q: How do you leave Detached HEAD?**
**A:** `git checkout <branch-name>` (e.g., `git checkout main`).

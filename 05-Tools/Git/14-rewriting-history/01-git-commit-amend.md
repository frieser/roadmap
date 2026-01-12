# Git Commit Amend

## Summary
The `--amend` flag allows you to modify the most recent commit. It combines the current staging area with the previous commit and creates a new commit (new SHA) to replace it.

## Detailed Explanation

### Usage
```bash
# Fix typo in message
git commit --amend -m "New correct message"

# Add a forgotten file
git add forgotten_file.go
git commit --amend --no-edit
```

### Safety
**Never** amend a commit that you have already pushed to a shared branch. Since it changes the commit hash, it causes history divergence for everyone else who pulled the old commit.

## Interview Questions
**Q: Can you amend an older commit (not the latest one)?**
**A:** Not directly with `--amend`. You would need to use Interactive Rebase (`git rebase -i`).

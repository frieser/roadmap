# Staging Area

## Summary
The Staging Area (also called the **Index** or **Cache**) is a file in the `.git` directory that stores information about what will go into your next commit. It acts as a preview or "holding zone" for changes, allowing you to craft atomic commits.

## Detailed Explanation
Unlike other VCS tools where you "commit all changes", Git forces you to explicitly add changes to the Staging Area.
*   `git add <file>`: Moves changes from Working Directory to Staging Area.
*   `git commit`: Takes whatever is in the Staging Area and wraps it into a commit.

### Why use a Staging Area?
1.  **Atomic Commits**: You can edit 10 files but only commit 2 of them that are related to a specific bug fix.
2.  **Review**: You can review exactly what you are about to commit using `git diff --staged`.
3.  **Partial Adding**: You can even stage parts of a file (patch mode: `git add -p`).

### Go-specific Context
Before staging Go code, it is best practice to format it.
```bash
# Workflow
go fmt ./...       # Format code in Working Directory
git add .          # Move formatted code to Staging Area
git commit -m "feat: add user login"
```
If you stage unformatted code, the commit will contain style violations.

## Interview Questions
**Q: How do you remove a file from the Staging Area but keep your local changes?**
**A:** Use `git reset HEAD <file>` or `git restore --staged <file>`. This un-stages the file but leaves the modifications in your Working Directory.

**Q: What is the difference between `git diff` and `git diff --staged`?**
**A:** `git diff` shows changes in the Working Directory that represent modifications not yet staged. `git diff --staged` shows changes that are staged and ready to be committed compared to the last commit.

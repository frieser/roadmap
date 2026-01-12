## Summary
**git reset --soft** moves the HEAD pointer to a previous commit, but leaves the **Index (Staging Area)** and **Working Directory** exactly as they were.

## Detailed Explanation
### **What happens**
- **HEAD**: Moves to the target commit.
- **Index**: Unchanged (contains the changes from the "undone" commits).
- **Working Directory**: Unchanged.

### **Use Case: Squashing Commits**
If you have 5 messy commits and want to combine them into one:
1. `git reset --soft HEAD~5`
2. `git commit -m "One clean commit message"`
All your changes from those 5 commits are still staged and ready to be committed as one.

## Interview Questions
**Q: When would you use `git reset --soft`?**
**A:** When you want to "undo" some commits but keep all the work you did in the staging area, usually to re-commit them with a better message or to squash multiple commits into one.

**Q: Does `git reset --soft` delete your code changes?**
**A:** No. It only moves the branch pointer. Your code changes remain in your staging area and your working directory.

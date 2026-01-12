## Summary
**git revert** creates a new commit that "undoes" the changes from a previous commit. It is a "safe" way to undo history because it doesn't rewrite existing commits, making it ideal for shared branches.

## Detailed Explanation
### **How it works**
If commit `A` added a line, `git revert A` will create a new commit `B` that removes that line. The history looks like: `A -> C -> D -> B(revert A)`.

### **AI Context**
- **Model Regression**: If you merge a new model architecture that significantly degrades performance, `git revert <merge-commit-hash>` is the fastest and safest way to return to the previous stable state on the `main` branch.
- **Data Leak**: If someone accidentally commits a small sample of private data, `git revert` can remove it from the HEAD, but **note**: the data still exists in the history. For true deletion, you need more advanced tools like BFG Repo-Cleaner.

## Interview Questions
**Q: What is the difference between `git revert` and `git reset`?**
**A:** `git revert` creates a NEW commit that reverses the changes of an old one, keeping the history intact. `git reset` moves the branch pointer back in time, effectively "deleting" commits from the branch's history.

**Q: Why is `git revert` preferred over `git reset` on a shared repository?**
**A:** Because `git revert` doesn't change existing history. If you use `git reset` on a shared branch, you will break the repository for everyone else who has already pulled those commits.

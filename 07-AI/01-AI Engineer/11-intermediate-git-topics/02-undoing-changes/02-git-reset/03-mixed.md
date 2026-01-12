## Summary
**git reset --mixed** (the default) moves the HEAD pointer and updates the **Index** to match the target commit, but leaves the **Working Directory** unchanged.

## Detailed Explanation
### **What happens**
- **HEAD**: Moves to the target commit.
- **Index**: Overwritten to match target (unstages your changes).
- **Working Directory**: Unchanged (your code is still there).

### **Use Case: Unstaging Changes**
If you accidentally ran `git add .` and staged some large model weights or private data:
`git reset HEAD large_file.bin` (which is a mixed reset for a specific path).
This removes the file from the next commit but keeps it on your disk.

## Interview Questions
**Q: What is the default mode of `git reset`?**
**A:** The default mode is `--mixed`.

**Q: What is the practical result of running `git reset HEAD~`?**
**A:** It "undoes" the last commit and "unstages" its changes. Your code remains in the working directory as "unstaged changes," allowing you to modify them and add them back selectively.

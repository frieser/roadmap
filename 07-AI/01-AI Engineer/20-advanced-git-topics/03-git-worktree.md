## Summary
`git worktree` allows you to have multiple working directories attached to the same repository. This enables you to work on multiple branches simultaneously without needing to stash your changes or clone the repository multiple times.

## Detailed Explanation
### **Why use Worktrees?**
In a traditional Git setup, you can only have one branch checked out at a time. Switching branches requires a clean working directory. Worktrees solve this by letting you checkout `branch-a` in folder A and `branch-b` in folder B.

### **Core Commands**
- **Add a worktree**: `git worktree add ../new-feature-dir feature-branch`
- **List worktrees**: `git worktree list`
- **Remove a worktree**: `git worktree remove ../new-feature-dir`

### **AI Engineering Use Case**
- **Parallel Experiments**: Run a long training job on `experiment-v1` in one worktree while developing `experiment-v2` in another.
- **Quick Bug Fixes**: If you are in the middle of a complex model refactor and a critical bug is reported in production, you can open a new worktree to fix the bug without touching your messy experimental code.

## Interview Questions
- **Q: What is the main advantage of `git worktree` over `git clone`?**
- **A:** Worktrees share the same `.git` directory, meaning they share the same local object database and configuration. This is much faster and uses less disk space than multiple clones.

- **Q: Can you have the same branch checked out in two different worktrees?**
- **A:** No, Git prevents this to avoid conflicts in the index and reference updates.

- **Q: How do you cleanup a worktree directory that was deleted manually?**
- **A:** Run `git worktree prune`.

# Linear vs Non-Linear History

## Summary
A linear history looks like a straight line of commits. A non-linear history looks like a tree with splitting and merging lines. Linear history is generally easier to read, while non-linear history preserves the exact context of how development happened.

## Detailed Explanation

### Linear History
*   **Achieved via**: Rebase workflows.
*   **Pros**: Easy to follow, `git bisect` works better.
*   **Cons**: Loses context of feature branches.

### Non-Linear History
*   **Achieved via**: Merge workflows (especially `--no-ff`).
*   **Pros**: Truthful representation of development.
*   **Cons**: Can become "spaghetti" (hard to follow).

### Go-specific Context
Many Go projects (like Kubernetes) enforce a linear history for the main branch to ensure that every commit is buildable and the timeline is clean.

## Interview Questions
**Q: How do you linearize a branch?**
**A:** `git rebase main`.

**Q: Why do some teams dislike merge commits?**
**A:** They create "clutter" in the log, making it harder to scan the history for specific changes.

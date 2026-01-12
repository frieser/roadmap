## Summary
A **Clean Git History** is a chronological record of changes that is easy to follow and understand. It involves techniques like rebasing, squashing, and writing clear commits to avoid a "spaghetti" history of merge commits and "fix typo" messages.

## Detailed Explanation
### **Techniques for a Clean History**
1. **Interactive Rebase (`git rebase -i`)**: Use this to rewrite your local history before pushing. You can combine (squash), rename, or delete commits.
2. **Squash on Merge**: When merging a PR, combine all commits into a single, well-described commit on the main branch. This hides the "trial and error" commits made during development.
3. **Avoid Unnecessary Merge Commits**: Use `git pull --rebase` to keep your local history linear when fetching changes from the remote.
4. **Atomic Commits**: Each commit should do **one thing**. Don't mix a feature implementation with a refactor of unrelated code.

### **Why it matters for AI Engineers**
- **Traceability**: If a model's performance drops, a clean history makes it easier to use `git bisect` to find the exact commit that caused the issue.
- **Reversibility**: It's much easier to revert a single "feature" commit than to find and revert 20 small, scattered commits.
- **Readability**: Senior engineers and researchers can quickly scan the history to understand the evolution of the model architecture.

## Interview Questions
**Q: What is the difference between `git merge` and `git rebase`?**
**A:** `git merge` creates a new "merge commit" that joins two histories, preserving the exact history of both branches. `git rebase` moves your changes on top of the target branch, creating a linear history without merge commits.

**Q: When is it unsafe to use `git rebase`?**
**A:** It is unsafe to rebase commits that have already been pushed to a shared remote repository, as it rewrites history and can cause major conflicts for other team members. Only rebase your own local, unpushed branches.

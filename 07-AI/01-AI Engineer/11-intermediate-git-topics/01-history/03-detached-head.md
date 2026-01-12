## Summary
A **Detached HEAD** state occurs when you check out a specific commit, tag, or remote branch instead of a local branch. In this state, HEAD points directly to a commit hash rather than a branch reference.

## Detailed Explanation
### **Why it happens**
- `git checkout <commit-hash>`
- `git checkout v1.0.0` (checking out a tag)
- `git checkout origin/main` (checking out a remote branch)

### **Risks for AI Engineers**
If you make commits in a detached HEAD state, they are not associated with any branch. If you switch back to another branch, these "orphan" commits can be lost and eventually deleted by Git's garbage collector.

### **How to fix it**
If you want to keep the changes you made while detached, create a new branch immediately:
`git checkout -b new-experimental-branch`

## Interview Questions
**Q: How do you get out of a detached HEAD state without losing your work?**
**A:** By creating a new branch (`git checkout -b <name>`). This "attaches" the current HEAD to a new branch reference, ensuring your commits are preserved.

**Q: What happens if you run `git checkout main` while in a detached HEAD state with uncommitted changes?**
**A:** Git will usually allow the checkout if the changes don't conflict with `main`. However, any *commits* you made while detached will stay "behind" and won't be visible on `main` unless you merge them.

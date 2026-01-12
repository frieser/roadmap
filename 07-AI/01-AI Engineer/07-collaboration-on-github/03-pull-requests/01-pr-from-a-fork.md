## Summary
A **Pull Request (PR) from a fork** is the standard way to contribute to a repository you don't have write access to. You create a fork, make changes in a branch, and then request the original repository's maintainers to "pull" your changes into their project.

## Detailed Explanation
For AI Engineers, this is how you contribute to major libraries like `PyTorch`, `TensorFlow`, or `Transformers`.

### **The Workflow**
1. **Fork**: Click "Fork" on the target repo.
2. **Clone**: Clone your fork locally (`git clone <your-fork-url>`).
3. **Branch**: Create a descriptive branch for your fix/feature (`git checkout -b fix/adamw-optimizer-bug`).
4. **Commit**: Make changes and commit them.
5. **Push**: Push the branch to *your* fork (`git push origin fix/adamw-optimizer-bug`).
6. **PR**: Go to the original repository on GitHub. GitHub will often show a "Compare & pull request" button.

### **AI-Specific Considerations**
- **Draft PRs**: If you are working on a complex model change, open a **Draft PR** early to get feedback on the approach before finishing the implementation.
- **Syncing**: Always sync your fork's main branch with the upstream before starting a new PR to avoid messy merge conflicts.
- **Test Results**: In the PR description, include logs or screenshots showing that your change improves model performance or fixes the bug without regressions.

## Interview Questions
**Q: What is the difference between a Pull Request and a Merge Request?**
**A:** They are effectively the same concept. "Pull Request" is the terminology used by GitHub and Bitbucket, while "Merge Request" is used by GitLab. Both describe the process of proposing changes to be merged into a branch.

**Q: Why should you create a new branch in your fork instead of using the main branch for a PR?**
**A:** Using a feature branch allows you to keep your fork's main branch clean and in sync with the upstream. It also enables you to work on multiple different PRs simultaneously from the same fork.

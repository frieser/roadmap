# Checking Out Tags

## Summary
Checking out a tag allows you to view the state of the repository at that specific point in time. This is essential for reproducing experiments or debugging issues in specific model versions.

## Detailed Explanation

### Basic Command
```bash
git checkout v1.0.0
```

### "Detached HEAD" State
When you check out a tag, you enter a "detached HEAD" state. This means you are not on a branch. If you make changes and commit them, they will not belong to any branch and will be hard to find later.

### Best Practice: Create a Branch from a Tag
If you need to fix a bug in an old version (e.g., `v1.0.0`), you should create a new branch from that tag:
```bash
git checkout -b fix-v1-bug v1.0.0
```

### AI Engineering Context
1.  **Reproducing Old Runs:** If a model trained 6 months ago (`v0.5`) is performing better than the current one, you can check out the `v0.5` tag to inspect the code and rerun the training.
2.  **Inference Debugging:** If a production model is failing, checking out the corresponding tag allows you to run local inference tests on the exact code used in production.

## Interview Questions
1.  **What does "detached HEAD" mean?**
    It means your HEAD is pointing directly to a specific commit hash rather than to a branch name.
2.  **How do you create a new branch starting from a specific tag?**
    `git checkout -b <new-branch-name> <tag-name>`
3.  **Can you "re-tag" a commit?**
    Not directly; you have to delete the old tag and create a new one with the same name pointing to the new commit.

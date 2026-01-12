---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Merging is the process of integrating changes from one branch into another. In an AI project, this typically happens when an experiment is successful and you want to bring those improvements (e.g., a better data loader or a new model layer) into the main codebase. Merging allows teams to combine individual contributions into a single, unified project history.

## Detailed Explanation

### The Basic Merge Workflow
Suppose you want to merge your `experiment` branch into `main`:

```bash
# 1. Switch to the receiving branch
git checkout main

# 2. Merge the source branch
git merge experiment-successful
```

### Types of Merges

#### 1. Fast-Forward Merge
Occurs if `main` hasn't changed since you created the `experiment` branch. Git simply moves the `main` pointer forward to match the `experiment` pointer. No new commit is created.

#### 2. Three-Way Merge (Recursive)
Occurs if `main` has moved forward with other commits while you were working on your experiment. Git creates a new **"merge commit"** that has two parent commits, representing the union of both histories.

### Handling Merge Conflicts
If you and a teammate modified the same line in `train.py`, Git won't know which version to keep.
1.  Git pauses the merge and marks the files as "conflicted."
2.  You must open the file, look for the `<<<<<<< HEAD` and `>>>>>>>` markers, and manually choose the correct code.
3.  Stage the resolved file with `git add` and finalize with `git commit`.

### AI Context: Merging Experiments
In AI engineering, merging isn't just about code; it's about results. Before merging an experiment:
*   **Verify Metrics**: Ensure the branch actually improves model performance.
*   **Check Configs**: Ensure the `.yaml` or `.json` settings in the branch are compatible with the main environment.
*   **Cleanup**: Remove any temporary debugging logs or local paths before merging.

## Interview Questions

**Q: What is a merge conflict?**
**A:** A merge conflict occurs when Git cannot automatically reconcile differences between two branches being merged. This usually happens when the same line in the same file has been modified in both branches.

**Q: How does a "Fast-Forward" merge differ from a "Three-Way" merge?**
**A:** A Fast-Forward merge happens when the target branch has no new commits since the source branch diverged; Git just moves the pointer. A Three-Way merge happens when both branches have diverged; Git creates a new "merge commit" to join the two histories.

**Q: How do you abort a merge if it becomes too complicated?**
**A:** Use `git merge --abort`. This will revert your repository to the state it was in before you started the merge, cleaning up any conflict markers.

**Q: Why should you pull the latest changes from `main` into your feature branch before merging back?**
**A:** By merging `main` into your feature branch first, you can resolve any conflicts in your isolated environment. This ensures that the final merge into `main` is clean and doesn't break the stable codebase for others.

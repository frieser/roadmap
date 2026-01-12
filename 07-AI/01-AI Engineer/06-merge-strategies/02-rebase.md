---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Rebasing is the process of moving or combining a sequence of commits to a new base commit. Instead of "merging" a branch (which creates a new joint commit), rebasing effectively "replays" your changes on top of another branch. For AI Engineers, rebasing is a powerful tool for maintaining a clean, linear project history and ensuring that their experimental code is always compatible with the latest updates in the main library.

## Detailed Explanation

### Rebase vs. Merge
*   **Merge**: Joins two histories. It's non-destructive and preserves the original timeline.
*   **Rebase**: Rewrites history. It takes your commits and "lifts" them to start at the latest tip of another branch.

### Why Rebase?
1.  **Cleaner History**: Eliminates unnecessary merge commits that can clutter the graph.
2.  **Up-to-Date Baseline**: If `main` has new commits that fix bugs in the data loader, rebasing your `experiment` branch onto `main` ensures your experiment benefits from those fixes immediately.

### Essential Rebase Commands

#### 1. Basic Rebase
```bash
# While on your feature branch
git fetch origin
git rebase origin/main
```

#### 2. Interactive Rebase (`-i`)
The "magic wand" of Git. It allows you to edit, delete, or combine commits.
```bash
git rebase -i HEAD~3
```
In the editor that opens, you can:
*   `pick`: Keep the commit.
*   `reword`: Change the commit message.
*   `squash`: Combine the commit with the previous one.
*   `drop`: Delete the commit.

### The Golden Rule of Rebasing
**Never rebase commits that have been pushed to a public repository.**
Since rebasing rewrites history (creates new commit hashes), it will break the workflow for everyone else who has pulled your original commits. Only rebase local commits that you haven't shared yet.

### AI Workflow Example
You've been working on a new "Attention" layer for 3 days. Meanwhile, `main` has been updated with a new version of PyTorch.
1.  `git checkout attention-layer`
2.  `git rebase main`
3.  Git "unplugs" your attention commits, updates the branch to the new PyTorch version, and "plugs" your commits back in.
4.  You resolve any conflicts and your branch is now perfectly up-to-date.

## Interview Questions

**Q: What is the primary difference between `git merge` and `git rebase`?**
**A:** `git merge` combines the work of two branches by creating a new merge commit, preserving the history as it happened. `git rebase` moves the entire feature branch so that it begins at the tip of the target branch, rewriting the commit history to be linear.

**Q: When is it dangerous to use `git rebase`?**
**A:** It is dangerous when used on commits that have already been pushed to a shared remote. Because rebase creates new commit hashes, it will cause "diverged history" for any teammate who has already based their work on your original commits.

**Q: What is an interactive rebase (`git rebase -i`)?**
**A:** It is a tool that allows you to modify your commit history before sharing it. You can reorder commits, change messages, "squash" multiple small commits into one clean one, or delete commits that were just temporary fixes.

**Q: How do you abort a rebase if it goes wrong?**
**A:** You can run `git rebase --abort`. This will stop the process and return your branch to exactly where it was before the rebase started.

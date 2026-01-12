---
tags: ['ai', 'roadmap', 'git']
---

## Summary
A **commit** is a permanent snapshot of your project's staged changes. In AI engineering, commits act as checkpoints for your experiment's progress, capturing the state of scripts, configuration files, and hyperparameters. Each commit is identified by a unique SHA-1 hash, creating an immutable history that allows you to trace exactly which code version produced a specific model result.

## Detailed Explanation

### The Anatomy of a Commit
When you run `git commit`, Git creates an object that contains:
1.  **Snapshot Reference**: A link to the "tree" (the state of all staged files).
2.  **Metadata**: Author, date, and timestamp.
3.  **Parent Pointer**: A reference to the previous commit (forming the history chain).
4.  **Message**: A human-readable description of the change.

### Essential Commit Commands
```bash
# Basic commit with a message
git commit -m "Update learning rate scheduler in train.py"

# Commit all tracked files (skips git add for already tracked files)
git commit -am "Refactor data augmentation pipeline"

# Amend the last commit (fix message or add forgotten changes)
git commit --amend
```

### Commit Messages in AI/ML Projects
A good commit message explains **why** a change was made. This is critical in AI projects where small tweaks can have massive impacts on performance.

**Bad Example**: `git commit -m "fixed stuff"`
**Good Example**: `git commit -m "Fix: adjust weight decay to 0.01 to prevent overfitting in late epochs"`

### Atomic Commits: The Gold Standard
Try to keep commits "atomic"—meaning they do one thing and do it well.
*   **Yes**: Commit 1: "Add ResNet-50 backbone", Commit 2: "Update DataLoader for multi-GPU".
*   **No**: Commit 1: "Added ResNet, fixed DataLoader, changed batch size, and updated README".

### AI Workflow Integration
*   **Experiment Tracking**: Many AI engineers include experiment IDs or metrics in their commit messages or use Git tags to mark significant milestones (e.g., `git tag v1.0-baseline`).
*   **Configuration Snapshots**: Always commit your `.yaml` or `.json` config files. Without the config, the code commit is often useless for reproducibility.

## Interview Questions

**Q: What is a SHA-1 hash in Git, and why is it important?**
**A:** A SHA-1 hash is a 40-character unique identifier for a commit. It is calculated based on the file contents, metadata, and the parent commit's hash. This ensures **data integrity**; if any part of the commit changes, the hash changes, making it impossible to secretly alter the project's history.

**Q: What is the difference between `git commit` and `git commit -a`?**
**A:** `git commit` only includes files that have been explicitly staged with `git add`. `git commit -a` automatically stages all **tracked** files that have been modified or deleted, then commits them. It does NOT include new (untracked) files.

**Q: When should you avoid using `git commit --amend`?**
**A:** You should never amend a commit that has already been pushed to a shared remote repository. Amending rewrites the history (creates a new hash), which will cause major conflicts for team members who have already pulled the original commit.

**Q: Explain how Git stores commits differently than systems that store "diffs."**
**A:** Systems like SVN store the differences (deltas) between versions. Git stores a **snapshot** of the entire project. If a file hasn't changed, Git doesn't store it again; it just stores a link to the previous version. This makes operations like switching branches or viewing history extremely fast.

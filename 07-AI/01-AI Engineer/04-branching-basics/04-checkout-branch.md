---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Checking out a branch is the act of switching your working directory to a different line of development. When an AI Engineer "checks out" a branch, Git updates the files on the disk to match the state of that branch's latest commit. This allows you to jump between different experimental setups—for example, switching from an NLP preprocessing branch to a computer vision training branch—in seconds.

## Detailed Explanation

### The Classic Command: `git checkout`
`git checkout` is a multi-purpose tool used to switch branches and restore files.
```bash
# Switch to an existing branch
git checkout experiment-v2

# Create and switch to a new branch (the -b flag)
git checkout -b new-experiment
```

### The Modern Alternative: `git switch`
In Git 2.23+, `git switch` was introduced to separate branch switching from file restoration.
```bash
# Switch to an existing branch
git switch main

# Create and switch to a new branch
git switch -c feature-api
```

### What Happens During Checkout?
1.  **HEAD Moves**: Git moves the `HEAD` pointer to the new branch.
2.  **Working Directory Updates**: Git replaces the files in your folder with the versions from the target branch.
3.  **Safety Check**: If you have unsaved (uncommitted) changes that would be overwritten by the switch, Git will stop you. You must either commit, stash, or discard those changes.

### AI Workflow: Switching Experiments
```bash
# Currently training on 'baseline'
# Suddenly you have an idea for a 'fast-attention' layer

# 1. Save current work
git add .
git commit -m "Save baseline state"

# 2. Switch to start the new idea
git checkout -b experiment-fast-attention

# ... code code code ...

# 3. Need to go back to main to fix a bug
git checkout main
```

## Interview Questions

**Q: What does it mean to be in a "Detached HEAD" state?**
**A:** This happens if you check out a specific commit hash or a tag instead of a branch name. You are no longer "on" a branch. Any commits you make in this state won't belong to any branch and will be hard to find later. To fix this, create a new branch from your current position: `git checkout -b recovery-branch`.

**Q: What happens if you try to switch branches while you have uncommitted changes?**
**A:** If the changes don't conflict with the target branch, Git may allow the switch and carry the changes over. If there is a conflict (the target branch has different changes in the same file), Git will block the switch and ask you to commit or stash your changes first.

**Q: How do you switch back to the previous branch you were on?**
**A:** Use the shortcut: `git checkout -`. This is very helpful for jumping back and forth between two branches (like `main` and your `feature` branch).

**Q: Why is `git switch` preferred over `git checkout` in newer versions of Git?**
**A:** `git checkout` is often confusing because it does too many things (switches branches, restores files, switches to specific commits). `git switch` is dedicated solely to branch management, making the command more intuitive and less prone to accidental file overwrites.

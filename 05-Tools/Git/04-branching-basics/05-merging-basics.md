# Merging Basics

## Summary
Merging is the way you integrate changes from one branch into another. Git automatically combines the work, but sometimes manual conflict resolution is required.

## Detailed Explanation

### Usage
To merge `feature` into `main`:
1.  Switch to the target branch: `git checkout main`
2.  Run merge: `git merge feature`

### Types of Merges
1.  **Fast-Forward**: If `main` has not diverged, Git just moves the pointer forward. No new commit is created.
2.  **Merge Commit (3-way)**: If both branches have diverged, Git creates a new "merge commit" with two parents, joining the histories.

### Go-specific Context
When merging in Go, conflicts often happen in `go.sum` (checksums).
*   **Resolution**: If `go.sum` has conflicts, you can often just delete the conflict markers, keep one version (or even delete the file), and run `go mod tidy` to regenerate it cleanly.

## Interview Questions
**Q: What is a "fast-forward" merge?**
**A:** It happens when the target branch is directly upstream of the current branch. Git simply moves the branch pointer forward to the latest commit without creating a dedicated merge object.

**Q: How do you abort a merge if it gets too messy?**
**A:** `git merge --abort`. This returns your repository to the state before you started the merge.

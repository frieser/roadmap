# Diff Between Branches

## Summary
Before merging a branch, it is crucial to see exactly what changes it introduces compared to your target branch.

## Detailed Explanation

### Usage
```bash
git diff main feature
```
This compares the tip of `main` with the tip of `feature`.

### Triple Dot Diff
```bash
git diff main...feature
```
This shows the changes that occurred on `feature` since it started/diverged from `main`. This is usually what you actually want to see (the "Pull Request" view).

## Interview Questions
**Q: What is the difference between `..` and `...` in git diff?**
**A:**
*   `git diff A..B`: Compares the tips of A and B directly.
*   `git diff A...B`: Compares B against the common ancestor of A and B (shows what happened on B since divergence).

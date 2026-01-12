# Unstaged Changes

## Summary
Unstaged changes are modifications in your Working Directory that have not yet been added to the Staging Area. This is the default view of `git diff`.

## Detailed Explanation

### Usage
```bash
git diff
```

### What it compares
It compares your **Working Directory** vs the **Index** (Staging Area).
If you haven't staged anything, it effectively compares Working Directory vs HEAD.

### Go-specific Context
If you ran `go fmt` but haven't staged it, `git diff` will show the formatting changes.

## Interview Questions
**Q: How do you ignore whitespace changes in diff?**
**A:** `git diff -w`. This is useful if you just re-indented code and want to see the logic changes.

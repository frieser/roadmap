# Checkout Tags

## Summary
Checking out a tag puts you in a "Detached HEAD" state at that specific commit. It is useful for building a specific version of software or reproducing a bug from an old release.

## Detailed Explanation

### Usage
```bash
git checkout v1.0.0
```
You are now looking at the code exactly as it was when v1.0.0 was released.

### Creating a branch from a tag
If you need to fix a bug in v1.0 (hotfix):
```bash
git checkout -b hotfix/v1.0.1 v1.0.0
```
This creates a new branch starting at the v1.0.0 tag.

## Interview Questions
**Q: Can you commit to a tag?**
**A:** No. Tags are immutable pointers. If you commit while checked out at a tag, you are just adding commits on top of that commit (creating a new history fork), but the tag itself stays where it was.

# Renaming a Branch

## Summary
You might need to rename a branch if the scope of your work changes or if you made a typo. The command is `git branch -m`.

## Detailed Explanation

### Renaming Local Branch
If you are on the branch you want to rename:
```bash
git branch -m <new-name>
```

If you are on a different branch:
```bash
git branch -m <old-name> <new-name>
```

### Renaming Remote Branch
Renaming a remote branch is actually a two-step process:
1.  Rename local branch.
2.  Delete the old remote branch.
3.  Push the new local branch.

```bash
git branch -m old-name new-name
git push origin --delete old-name
git push origin new-name
```

## Interview Questions
**Q: How do you rename the master branch to main?**
**A:** `git branch -m master main`. Then you need to update the upstream tracking if you push it.

**Q: Does renaming a branch affect the commit history?**
**A:** No, the history (commits) remains exactly the same. Only the pointer (branch name) changes.

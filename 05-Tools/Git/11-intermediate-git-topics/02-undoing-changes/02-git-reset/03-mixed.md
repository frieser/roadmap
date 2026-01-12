# Git Reset Mixed

## Summary
`git reset --mixed` is the default mode if no flag is provided. It moves HEAD to a previous commit and keeps changes in the Working Directory but **unstages** them.

## Detailed Explanation

### Usage
```bash
git reset HEAD~1
# Equivalent to
git reset --mixed HEAD~1
```

### Scenario
You committed some work, but now you realize the work is incomplete. You want to keep the code, but you don't want it staged for commit yet.
1.  `git reset HEAD~1`.
2.  Commit is undone.
3.  Changes are in Working Directory (Modified, not Staged).
4.  You can continue editing.

## Interview Questions
**Q: What is the difference between Soft and Mixed?**
**A:** Soft leaves changes **Staged**. Mixed leaves changes **Unstaged**.

**Q: Is Mixed reset destructive?**
**A:** No, your file contents in the working directory are preserved.

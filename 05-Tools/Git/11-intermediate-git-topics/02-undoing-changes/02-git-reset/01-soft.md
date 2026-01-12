# Git Reset Soft

## Summary
`git reset --soft` moves the HEAD pointer to a previous commit, but keeps your changes staged (in the index).

## Detailed Explanation

### Usage
```bash
git reset --soft HEAD~1
```

### Scenario
You just ran `git commit` but realized you forgot to add one file, or you want to fix the commit message completely.
1.  `git reset --soft HEAD~1` (Undo the commit action).
2.  Changes are now back in Staging Area.
3.  Add the missing file.
4.  `git commit` again.

(Note: `git commit --amend` is easier for this specific case, but soft reset is more flexible if you want to split one commit into two).

## Interview Questions
**Q: What happens to the working directory in a soft reset?**
**A:** Nothing. The files in your working directory are untouched.

**Q: Where do the changes go?**
**A:** They stay in the Staging Area (Index), ready to be committed again.

# Git Reset Hard

## Summary
`git reset --hard` is a destructive command. It moves HEAD to a previous commit and **destroys** all changes in the Staging Area and Working Directory to match that commit.

## Detailed Explanation

### Usage
```bash
git reset --hard HEAD~1
```
This effectively deletes the last commit and all work associated with it.

### Usage 2 (Cleanup)
```bash
git reset --hard origin/main
```
Force your local branch to match the remote branch exactly, discarding all local commits and changes.

### Danger Zone
**Warning**: You cannot undo a hard reset easily (unless you use `git reflog` immediately). Any uncommitted changes in the working directory are lost forever.

## Interview Questions
**Q: When should you use `--hard`?**
**A:** When you want to scrap your current work entirely and start over from a clean state.

**Q: Does `git reset --hard` affect untracked files?**
**A:** No, usually it only touches tracked files. To remove untracked files, you need `git clean -fd`.

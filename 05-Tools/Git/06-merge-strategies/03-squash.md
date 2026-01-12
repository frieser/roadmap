# Squash

## Summary
Squashing involves combining multiple commits into a single commit. This is useful for cleaning up a messy history (e.g., "wip", "typo", "fix") into one clean "feat: complete login" commit before merging.

## Detailed Explanation

### Squash via Merge
When merging a Pull Request, you can "Squash and Merge".
```bash
git merge --squash feature
```
This takes all changes from `feature`, stages them, but doesn't commit. You then make one final commit.

### Squash via Interactive Rebase
```bash
git rebase -i HEAD~N
# Change 'pick' to 'squash' (or 's') for the commits you want to meld.
```

### Go-specific Context
Go modules rely on semantic versions. Having 50 commits for one feature makes `git bisect` harder. Squashing ensures that every commit on `main` builds and passes tests (`go test ./...`), which is crucial for maintaining a healthy codebase.

## Interview Questions
**Q: What happens to the intermediate commit messages when you squash?**
**A:** Git usually concatenates them into the body of the new single commit message, giving you a chance to edit/summarize them.

**Q: Can you un-squash?**
**A:** Only if you haven't pushed or if you can find the old commits in `git reflog`. Once rewritten and garbage collected, the old individual commits are gone.

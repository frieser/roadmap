# Git Reflog

## Summary
The Reflog (Reference Log) is your safety net. It records every time the tip of a branch (HEAD) is updated. If you accidentally delete a branch or lose a commit via hard reset, it is almost certainly still in the reflog.

## Detailed Explanation

### Usage
```bash
git reflog
```
Output:
```
a1b2c3d HEAD@{0}: commit: fix bug
e5f6g7h HEAD@{1}: checkout: moving from main to feature
```

### Recovering Lost Work
If you accidentally `git reset --hard HEAD~1` and lost a commit:
1.  Run `git reflog`.
2.  Find the SHA of the commit before the reset (e.g., `a1b2c3d`).
3.  `git reset --hard a1b2c3d`.

### Go-specific Context
Useful if you mess up a `go mod tidy` or `git rebase` and want to go back to exactly how the repo looked 5 minutes ago.

## Interview Questions
**Q: Does reflog persist forever?**
**A:** No, entries typically expire after 90 days (garbage collection). Also, it is strictly local; pushing does not send your reflog to the server.

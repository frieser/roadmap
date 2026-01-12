# Git Bisect

## Summary
`git bisect` uses a binary search algorithm to find which commit introduced a bug. It is incredibly efficient for finding regressions in a long history.

## Detailed Explanation

### Workflow
1.  Start: `git bisect start`.
2.  Mark bad: `git bisect bad` (Current broken version).
3.  Mark good: `git bisect good v1.0` (Last known working version).
4.  Git checks out a commit halfway between.
5.  You test. If broken, run `git bisect bad`. If working, run `git bisect good`.
6.  Repeat until Git says: "a1b2c3d is the first bad commit".

### Go-specific Context
You can automate this with `go test`:
```bash
git bisect run go test ./...
```
Git will automatically checkout, run the test, and mark it good/bad until it finds the culprit. This is a superpower for Go developers.

## Interview Questions
**Q: What if the code doesn't compile on one of the intermediate commits?**
**A:** You can use `git bisect skip` to move to an adjacent commit.

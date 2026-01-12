# Cherry Picking

## Summary
`git cherry-pick` allows you to apply the changes introduced by some existing commits (from another branch) onto your current branch. It copies the commit.

## Detailed Explanation

### Usage
```bash
git cherry-pick <commit-sha>
```
You can pick multiple commits:
```bash
git cherry-pick <sha-A> <sha-B>
```

### Use Cases
1.  **Backporting**: Applying a bug fix from `main` to an older `v1.0` release branch.
2.  **Mistake Correction**: You committed to the wrong branch. You switch to the right branch, cherry-pick the commit, and then reset the wrong branch.

### Go-specific Context
Commonly used in maintaining older versions of Go libraries.
*   Bug fix `fix: nil pointer exception` lands in `main` (v2.0 dev).
*   Release manager cherry-picks that specific commit to `release/v1.5` branch to ship a patch release.

## Interview Questions
**Q: Does cherry-pick move the commit?**
**A:** No, it copies the *changes* and creates a *new* commit with a *new* SHA hash on your current branch. The original commit stays where it was.

**Q: What happens if there is a conflict during cherry-pick?**
**A:** The process pauses. You resolve conflicts, `git add`, and then `git cherry-pick --continue`.

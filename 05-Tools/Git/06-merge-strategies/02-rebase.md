# Rebase

## Summary
Rebasing is the process of moving or combining a sequence of commits to a new base commit. It rewrites history to create a linear progression, as if you had started your work from the latest version of the main branch.

## Detailed Explanation

### Workflow
You are on `feature` branch. `main` has moved forward.
```bash
git checkout feature
git rebase main
```
Git essentially:
1.  Saves your commits to a temporary area.
2.  Resets your branch to `main`.
3.  Re-applies your commits one by one on top of the new `main`.

### Interactive Rebase
A powerful tool to clean up local history before pushing.
```bash
git rebase -i HEAD~3
```
Allows you to:
*   **pick**: Keep commit.
*   **reword**: Change message.
*   **squash**: Merge into previous commit.
*   **drop**: Delete commit.

### Go-specific Context
**Golden Rule**: Never rebase public branches (like `main`) that others base their work on. Only rebase your local feature branches.
In Go teams, "rebase and merge" is a common strategy to ensure that the `main` branch stays linear and tests pass on the exact state of code that is merged.

## Interview Questions
**Q: What is the main danger of rebasing?**
**A:** It rewrites history (changes SHA-1 hashes). If you rebase commits that you have already pushed and others have pulled, you will break their work.

**Q: How do you solve conflicts during a rebase?**
**A:** Fix the files, then `git add <file>` and `git rebase --continue`. Do not run `git commit`.

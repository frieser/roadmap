---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Cherry-picking is the process of selecting a specific commit from one branch and applying it to another. Unlike merging or rebasing, which bring over entire histories, cherry-picking is surgical. For AI Engineers, this is useful when you want to port a single bug fix or a specific hyperparameter optimization from an experimental branch into the stable `main` branch without bringing along all the other unfinished changes from that experiment.

## Detailed Explanation

### The Command
```bash
git cherry-pick <commit_hash>
```

### When to Cherry-Pick in AI Projects
1.  **Hotfixes**: You're working on a long-term "Transformer-v3" experiment and you find a critical bug in the data loader. You fix it in your experiment branch, but you want that fix in `main` immediately without merging the unfinished model.
2.  **Specific Wins**: An experiment failed overall, but one specific commit—perhaps a new data augmentation technique—showed great results. You "cherry-pick" that success into your baseline.
3.  **Recovery**: Accidental commits to the wrong branch can be "moved" by cherry-picking them to the correct branch and then deleting them from the wrong one.

### How it Works
Git calculates the diff (change) introduced by the specific commit and tries to apply that exact same change to your current branch. It creates a **new commit** with a different hash but the same content and message.

### Potential Issues
*   **Duplicate Commits**: If you later merge the original branch, Git might see the "same" change twice. Modern Git handles this well, but it can occasionally lead to conflicts.
*   **Dependency Gaps**: If the commit you are cherry-picking relies on code from a previous commit that you *didn't* cherry-pick, you will get a merge conflict.

## Interview Questions

**Q: What is `git cherry-pick`?**
**A:** It is a command that allows you to pick a single commit from any branch and apply it to your current branch as a new commit. It is used for "surgical" integration of specific changes.

**Q: When would an AI Engineer use cherry-pick instead of merge?**
**A:** They would use it when they only want one specific change (like a bug fix or a successful hyperparameter tweak) from a feature branch, but the rest of the work on that branch is not yet ready or should not be merged into the stable codebase.

**Q: What happens if you cherry-pick a commit and it causes a conflict?**
**A:** Git will pause, just like in a merge. You must resolve the conflict manually, `git add` the resolved file, and then run `git cherry-pick --continue` to finish the operation.

**Q: Does cherry-picking move the original commit?**
**A:** No. It stays in its original branch. Cherry-picking creates a **copy** of the change and applies it as a new commit on your current branch.

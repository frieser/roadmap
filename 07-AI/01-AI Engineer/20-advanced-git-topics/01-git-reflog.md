## Summary
The Git Reflog is a local record of where the tips of branches and other references have pointed in the past. It is the "ultimate safety net" in Git, allowing developers to recover commits that have been lost due to forced pushes, deleted branches, or accidental resets.

## Detailed Explanation
### **How Reflog Works**
Every time a reference (like `HEAD`) is updated, Git records the new and old values in the reflog. This is a purely local log; it is not pushed to the server or shared with others.
- **View Reflog**: `git reflog`
- **Recovering a lost commit**: If you accidentally ran `git reset --hard HEAD~1` and lost a commit, you can find the previous state in the reflog and restore it with `git reset --hard HEAD@{1}`.

### **AI Engineering Use Case**
During model experimentation, you might hard reset your branch to a "known good" state but later realize you lost a valuable hyperparameter change. `git reflog` allows you to jump back to that experimental commit even if it's no longer part of your branch history.

### **Common Commands**
- `git reflog show`: Displays the history of the current branch.
- `git reflog show <branch-name>`: Displays the history of a specific branch.
- `git checkout HEAD@{n}`: Detach HEAD at a specific point in time recorded in the reflog.

## Interview Questions
- **Q: Is the git reflog shared with other developers when you push?**
- **A:** No, the reflog is strictly local to your repository.

- **Q: How does `git reflog` differ from `git log`?**
- **A:** `git log` shows the public commit history of a branch. `git reflog` shows the local history of where your references (like HEAD) have pointed, including commits that might have been "lost" or deleted.

- **Q: How long does Git keep reflog entries?**
- **A:** By default, reachable entries are kept for 90 days, and unreachable entries are kept for 30 days (controlled by `gc.reflogExpire` settings).

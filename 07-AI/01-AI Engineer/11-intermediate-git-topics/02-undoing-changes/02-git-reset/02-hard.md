## Summary
**git reset --hard** is a destructive command that moves the HEAD pointer and overwrites both the **Index** and the **Working Directory** to match the target commit.

## Detailed Explanation
### **What happens**
- **HEAD**: Moves to the target commit.
- **Index**: Overwritten to match target.
- **Working Directory**: Overwritten to match target. **All uncommitted changes are lost forever.**

### **AI Context**
- **Total Failure**: Use this when an experiment has gone completely wrong, the environment is corrupted, and you just want to go back to exactly how things were at the last stable commit.
- **Warning**: Never use this if you have uncommitted research notes or experimental code that you haven't backed up elsewhere.

## Interview Questions
**Q: Why is `git reset --hard` considered dangerous?**
**A:** Because it permanently overwrites all changes in your working directory and staging area. If you haven't committed your work, Git has no way to recover it after a hard reset.

**Q: Can you ever recover work after a `git reset --hard`?**
**A:** If the work was **committed** at some point, you can find the commit hash using `git reflog` and reset back to it. If the work was never committed, it is generally unrecoverable.

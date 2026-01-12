## Summary
**Linear history** is a straight line of commits without merge commits, often achieved via rebasing. **Non-linear history** preserves the exact branching and merging structure, resulting in a complex graph. For AI Engineers, linear history is often preferred for readability, while non-linear is used to preserve the context of feature development.

## Detailed Explanation
### **Linear History**
- **Pros**: Easy to read, `git bisect` works perfectly, looks cleaner.
- **Cons**: Rewrites history (can be dangerous), loses the "context" of when a branch was started and finished.
- **AI Context**: Useful for production releases where you want a clean list of changes to model versions.

### **Non-Linear History**
- **Pros**: Accurate record of all development activity, easy to see which commits belonged to which feature.
- **Cons**: Can become a "spaghetti" of merge lines, harder to find when a specific bug was introduced.
- **AI Context**: Common in research environments where multiple experiments are running in parallel and eventually merged into a main project.

## Interview Questions
**Q: How do you achieve a linear history in Git?**
**A:** By using `git rebase` instead of `git merge` when integrating changes from a feature branch, and by using `git pull --rebase` to update your local branch from the remote.

**Q: When should you avoid forcing a linear history?**
**A:** Avoid it on shared public branches. Rewriting history on a branch that others are working on will cause major synchronization issues for the team.

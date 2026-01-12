# Git Rebase (Rewriting History)

## Summary
Rebase is the primary tool for rewriting history. While commonly used to update a branch (`git rebase main`), it is also used to linearize local history before merging.

## Detailed Explanation

### Git Pull Rebase
```bash
git pull --rebase origin main
```
This fetches the remote changes and replays your local unpushed commits on top of them. This avoids the "useless" merge commit that happens when you `git pull` and have diverged slightly.

### Rebase vs Merge
*   **Merge**: Preserves history exactly as it happened. Good for audit trails.
*   **Rebase**: Cleans history to look linear. Good for understanding the logical progression of the project.

### Go-specific Context
`go.sum` conflicts are easier to resolve during a rebase (one commit at a time) than in a massive merge commit where multiple dependency changes collide at once.

## Interview Questions
**Q: If you have a conflict during rebase, how do you finish?**
**A:** Resolve conflict -> `git add` -> `git rebase --continue`.

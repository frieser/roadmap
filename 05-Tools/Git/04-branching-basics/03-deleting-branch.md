# Deleting a Branch

## Summary
Once a feature is merged, the branch should be deleted to keep the repository clean. You can delete local and remote branches separately.

## Detailed Explanation

### Deleting Local Branch
*   **Safe Delete**: `git branch -d <branch-name>` (Only works if branch is merged).
*   **Force Delete**: `git branch -D <branch-name>` (Deletes even if unmerged - use with caution).

### Deleting Remote Branch
```bash
git push origin --delete <branch-name>
```

### Go-specific Context
In a team using Go modules, deleting old branches is important because `go.sum` conflicts can occur if long-lived branches diverge too much. Keep branches short-lived and delete them after merge.

## Interview Questions
**Q: Why would `git branch -d` fail?**
**A:** It fails if the branch has changes that have not yet been merged into your current branch (or upstream). This prevents accidental data loss.

**Q: How do you prune local tracking branches that no longer exist on remote?**
**A:** `git fetch -p` (or `--prune`).

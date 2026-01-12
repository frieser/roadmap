# Managing Remotes

## Summary
A remote is a common repository that all team members use to exchange their changes. The default remote is usually called `origin`. Managing remotes involves adding, removing, or changing the URLs of these remote repositories.

## Detailed Explanation

### Commands
*   **List remotes**: `git remote -v` (shows URL).
*   **Add remote**: `git remote add <name> <url>`.
*   **Remove remote**: `git remote remove <name>`.
*   **Rename remote**: `git remote rename <old> <new>`.

### Origin vs Upstream
*   **Origin**: Your fork or the main repo you cloned from.
*   **Upstream**: The original repo you forked from (used to sync your fork with the original project).

### Go-specific Context
When forking a Go package on GitHub to submit a PR:
1.  Clone your fork: `git clone git@github.com:you/repo.git` (Remote: `origin`).
2.  Add original repo: `git remote add upstream git@github.com:original/repo.git`.
3.  Pull latest changes from `upstream` to keep your local dev environment current.

## Interview Questions
**Q: What is `origin` in Git?**
**A:** `origin` is just the default nickname Git gives to the server you cloned from. It has no special meaning other than convention.

**Q: How do you change the URL of a remote (e.g., switching from HTTPS to SSH)?**
**A:** `git remote set-url origin git@github.com:user/repo.git`.

# Git Push Force

## Summary
`git push --force` allows you to overwrite the remote repository with your local history, even if it causes the remote to lose commits. It is required after rewriting history (amend, rebase).

## Detailed Explanation

### The Danger
If teammate A pushed a commit, and you force push your branch which doesn't have A's commit, A's work is lost on the server.

### Force with Lease (The Safer Way)
```bash
git push --force-with-lease
```
This checks: "Is the remote value what I think it is?". If someone else pushed changes that you haven't fetched yet, the push will fail. This prevents accidental overwrites of team members' work.

## Interview Questions
**Q: When is force pushing acceptable?**
**A:** On a personal feature branch that only you are working on, or after a rebase that was agreed upon by the team. Never on `main`.

# Git Filter Branch

## Summary
`git filter-branch` is a powerful (but slow and complex) command to rewrite huge swathes of history. It is often used to remove a file (like a password or large binary) from every commit in the repository's past.

## Detailed Explanation

### Usage (Deprecated)
The command is difficult to use correctly.
```bash
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
```

### Modern Alternatives
The Git project now recommends using **`git-filter-repo`** (a Python-based tool) or **BFG Repo-Cleaner**. They are faster and safer.

### Use Cases
1.  **Extracting a subdirectory**: Turning a folder into its own repository, preserving history for just that folder.
2.  **Redacting secrets**: Permanently deleting an accidentally committed API key.

## Interview Questions
**Q: After using filter-branch, do the commit hashes change?**
**A:** Yes, every single commit hash from the point of change forward is recalculated. You must force push (`git push -f`) to update the remote.

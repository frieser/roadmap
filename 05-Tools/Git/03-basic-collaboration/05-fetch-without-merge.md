# Fetch without Merge

## Summary
`git fetch` downloads commits, files, and refs from a remote repository into your local repository but **does not** merge them into your working files. It is safe and non-destructive.

## Detailed Explanation
When you run `git pull`, Git fetches and then immediately attempts to merge. `git fetch` allows you to see what others have done before integrating it.

### Workflow
1.  **Fetch**: `git fetch origin` (Updates `origin/main` pointer).
2.  **Inspect**: `git log origin/main` (See what's new).
3.  **Diff**: `git diff main origin/main` (See changes).
4.  **Merge**: `git merge origin/main` (If you are happy).

### Go-specific Context
Useful in CI/CD pipelines or when working with sensitive Go modules updates. You might fetch to check if `go.mod` has changed in a way that conflicts with your local work before deciding to merge.

## Interview Questions
**Q: What is the difference between `git fetch` and `git pull`?**
**A:** `git fetch` only updates the remote-tracking branches (like `origin/main`). It does not touch your working directory. `git pull` does a fetch followed by a merge into your current branch.

**Q: Where are the fetched changes stored?**
**A:** They are stored in the `.git/objects` database, and the remote references (like `.git/refs/remotes/origin/main`) are updated to point to the new commits.

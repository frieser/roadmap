# Committing Changes

## Summary
A commit is a snapshot of your repository at a specific point in time. It saves the changes currently in the Staging Area to the local repository history. Each commit has a unique SHA-1 hash, an author, a timestamp, and a message describing the change.

## Detailed Explanation
Committing is the action of saving your work.
```bash
git commit -m "Descriptive message"
```

### Commit Anatomy
*   **Tree Object**: Represents the directory structure/file blobs.
*   **Parent(s)**: The previous commit(s) this commit is based on.
*   **Author/Committer**: Who made the change and who committed it.
*   **Message**: Human-readable description.

### Best Practices
*   **Atomic**: One task per commit.
*   **Descriptive Messages**: Use the imperative mood ("Fix bug" not "Fixed bug").
*   **Never Commit Broken Code**: Ensure the project builds before committing.

### Go-specific Context
In Go projects, it's common to use **Conventional Commits** to automate versioning and changelogs.
```bash
# Example conventional commit
git commit -m "feat(api): add new endpoint for user profile"
```
Since Go modules use semantic versioning (`v1.0.0`), tools can analyze these commit messages to determine if a version bump should be Major, Minor, or Patch.

## Interview Questions
**Q: How do you modify the last commit message?**
**A:** `git commit --amend -m "New message"`. (Only do this if you haven't pushed the commit yet!)

**Q: What happens if you commit without running `git add`?**
**A:** Git will error out saying "nothing added to commit", unless you use the `-a` flag (`git commit -a -m "msg"`) which stages tracked files automatically.

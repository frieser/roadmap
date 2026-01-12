# Working Directory

## Summary
The Working Directory (or Working Tree) is the visible file system where you do your work. It contains the checkout of a specific version of the project. It is one of the "Three Trees" in Git, the others being the Staging Area (Index) and the Repository (HEAD).

## Detailed Explanation
When you checkout a branch, Git populates your **Working Directory** with the files from that snapshot.
*   **Untracked files**: New files you created but haven't added to Git yet.
*   **Modified files**: Tracked files you have changed but not yet staged.
*   **Unmodified files**: Files that match the version in the repository.

You edit files in the Working Directory. Git does not track these changes until you explicitly tell it to via `git add`.

### The Three States
1.  **Modified**: Changed in Working Directory.
2.  **Staged**: Marked for next commit (Index).
3.  **Committed**: Safely stored in database (Repository).

### Go-specific Context
In Go, your Working Directory typically contains your `go.mod`, `*.go` source files, and tests.
The Go toolchain (`go build`, `go test`) operates on the files in your Working Directory. It doesn't know about Staged or Committed files—it only sees what's on the disk.

## Interview Questions
**Q: If you delete a file in the Working Directory, is it deleted from Git?**
**A:** Not immediately. It is just a change in the Working Directory. You must also `git rm` (stage the deletion) and commit it for Git to record the deletion.

**Q: How do you discard changes in the Working Directory (revert to last commit)?**
**A:** `git checkout -- <file>` (old way) or `git restore <file>` (new way). This overwrites the file in the Working Directory with the version from the Staging Area/HEAD.

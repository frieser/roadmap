# HEAD

## Summary
HEAD is a special pointer in Git that refers to the snapshot of your repository that you currently have checked out in your working directory.

## Detailed Explanation
*   Usually, HEAD points to a **Branch** name (e.g., `main`).
*   That Branch name points to a specific **Commit** SHA.
*   Therefore, HEAD indirectly points to the latest commit of your current branch.

When you make a new commit, the HEAD moves forward to point to the new commit (and the branch pointer moves with it).

### Go-specific Context
When you run `go build`, the compiler uses the files in your working directory (which HEAD represents).

## Interview Questions
**Q: Where is HEAD stored?**
**A:** In the `.git/HEAD` file. It's a text file containing `ref: refs/heads/branch-name`.

**Q: What is HEAD^?**
**A:** The parent of the current commit. `HEAD~2` is the grandparent.

# Git Filter-Branch & BFG Repo-Cleaner

## Summary
`git filter-branch` is a powerful but legacy tool for rewriting large parts of Git history. It is most commonly used to remove sensitive data (API keys) or large files (model weights) from the entire history of a repository. Note: Modern Git documentation recommends using `git-filter-repo` or `BFG Repo-Cleaner` instead.

## Detailed Explanation
Git history is immutable by default. Deleting a file in a new commit doesn't remove it from the history of previous commits. `filter-branch` (or its successors) iterates through every commit and applies a filter.

### Common Use Cases
- **Removing a file from all history:**
  ```bash
  git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
  ```
- **Extracting a subdirectory into a new repo:**
  ```bash
  git filter-branch --subdirectory-filter src/models --all
  ```

### AI Engineering Context
1.  **Removing Accidental Data Commits:** If someone accidentally committed a 10GB `.csv` file, deleting it in the next commit still leaves the repo size at 10GB+. You must use a history rewriter to purge it.
2.  **Sanitizing Public Releases:** Before open-sourcing a research repo, you might need to strip out internal paths, private server URLs, or proprietary weights that were committed early in the project.
3.  **Splitting Monorepos:** If an ML project grows too large, you might want to move the `preprocessing` scripts into their own repository while preserving their individual commit history.

### Modern Alternatives
- **BFG Repo-Cleaner:** Much faster and simpler than `filter-branch`.
- **git-filter-repo:** The currently recommended tool by the Git team.

## Interview Questions
1.  **Why is `git rm file.zip` followed by `git commit` not enough to reduce repository size?**
    The file remains in the history of previous commits. To reduce size, the file must be removed from ALL commits where it existed.
2.  **What are the risks of using `filter-branch` on a shared repo?**
    It rewrites EVERY commit hash. Everyone on the team will have to re-clone or perform a complex "hard reset" to the new history.
3.  **How do you remove a sensitive API key from history?**
    Use a tool like `BFG Repo-Cleaner` or `git-filter-repo` to search and replace the string or remove the file containing it across all commits.

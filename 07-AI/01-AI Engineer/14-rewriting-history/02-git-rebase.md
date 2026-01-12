# Git Rebase

## Summary
Rebasing is the process of moving or combining a sequence of commits to a new base commit. It is often used as an alternative to merging to maintain a clean, linear project history. For AI Engineers, this is vital for keeping feature branches up-to-date with a fast-moving `main` branch.

## Detailed Explanation
Unlike `git merge`, which creates a "merge commit," `git rebase` effectively "re-plays" your changes on top of another branch.

### Common Workflows
1.  **Update feature branch with main:**
    ```bash
    git checkout feature-experiment
    git rebase main
    ```
2.  **Interactive Rebase (Cleaning up history):**
    ```bash
    git rebase -i HEAD~5
    ```
    This allows you to `pick`, `squash`, `edit`, or `drop` commits.

### AI Engineering Context
1.  **Squashing Experiments:** You might have 20 "checkpoint" commits like "try new lr", "fix bug", "another test". Before merging into `main`, you can use `rebase -i` to squash these into a single, clean commit: "Implement Focal Loss for imbalanced data".
2.  **Syncing with Base Pipelines:** If the data engineering team updates the shared data loading library in `main`, you should rebase your model development branch on `main` to ensure your experimental code works with the latest pipeline.
3.  **Avoiding Merge Commits:** Many high-performance ML teams prefer linear histories to make `git bisect` (finding which commit introduced a bug) much easier.

### Rebase vs. Merge
- **Merge:** Preserves exact history, including "messy" intermediate steps. Non-destructive.
- **Rebase:** Creates a clean, linear story. Destructive (rewrites hashes).

## Interview Questions
1.  **What is the "Golden Rule" of rebasing?**
    Never rebase branches that have been pushed to a public/shared repository.
2.  **How do you handle a conflict during a rebase?**
    Fix the conflict, run `git add <file>`, and then run `git rebase --continue`. Do NOT run `git commit`.
3.  **When would you prefer `git merge` over `git rebase`?**
    When you want to preserve the complete history of how a feature evolved, or when you are working on a long-running shared branch where rebasing would disrupt others.

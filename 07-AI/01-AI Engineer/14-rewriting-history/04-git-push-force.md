# Git Push --Force

## Summary
`git push --force` (or `-f`) is used to overwrite the remote history with your local history. It is required after any operation that rewrites history (amend, rebase, filter-branch) because the remote and local branches have diverged.

## Detailed Explanation
Standard pushes only allow "fast-forward" updates (where your new commits are direct descendants of the remote commits). If you've rewritten history, your local commits have different hashes, so Git requires a force to overwrite.

### The Command
- **Dangerous Force:**
  ```bash
  git push origin feature-branch --force
  ```
- **Safer Force (Recommended):**
  ```bash
  git push origin feature-branch --force-with-lease
  ```
  `--force-with-lease` will fail if someone else has pushed new commits to the remote that you haven't pulled yet.

### AI Engineering Context
1.  **Experimental Branch Cleanup:** After squashing your intermediate "checkpoint" commits into a clean feature commit, you must force push to update your remote branch.
2.  **Recovering from Bad Commits:** If a broken model config was pushed to a feature branch, and you fixed it via `amend`, a force push updates the PR.
3.  **CI/CD Triggers:** Sometimes you might need to re-run a pipeline by force-pushing the same state (though this is often better handled via the CI UI).

### Best Practices
- **Only force push to your own feature branches.**
- **NEVER force push to `main` or `production`** (unless in an absolute emergency with team consensus).
- **Use `--force-with-lease`** to avoid accidentally overwriting a colleague's work.

## Interview Questions
1.  **What is the difference between `--force` and `--force-with-lease`?**
    `--force` overwrites the remote regardless of its state. `--force-with-lease` only overwrites if the remote branch is in the same state as your last `fetch`, preventing you from overwriting commits you haven't seen.
2.  **When is it acceptable to use `git push --force`?**
    On a private feature branch that you are the sole contributor for, after you have cleaned up your local history via rebase or amend.
3.  **How do you fix a situation where a colleague force-pushed to a branch you were working on?**
    You usually need to fetch the new remote state and then `git reset --hard origin/<branch>` (warning: this loses your local changes) or perform a complex rebase.

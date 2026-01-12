# Git Commit --Amend

## Summary
The `git commit --amend` command allows you to modify the most recent commit. This is useful for fixing typos in the commit message or adding forgotten files without creating a new, separate commit.

## Detailed Explanation
When you run `amend`, Git takes the staged changes and the last commit, and creates a *new* commit that replaces the old one.

### Basic Usage
1.  **Change the last commit message:**
    ```bash
    git commit --amend -m "New and better commit message"
    ```
2.  **Add a forgotten file to the last commit:**
    ```bash
    git add forgotten_file.py
    git commit --amend --no-edit
    ```

### AI Engineering Context
1.  **Polishing Experiment Logs:** If you committed a new model architecture but forgot to include the updated `requirements.txt`, `amend` keeps your history clean.
2.  **Release Cleanup:** When preparing a model for production, you might want to ensure the final commit message contains the correct version number or model hash.
3.  **Correcting Configs:** You committed a training run but realized you had the `debug` flag set to `True`. You can fix the flag, stage it, and `amend` the commit.

### CRITICAL WARNING
**Never amend a commit that has already been pushed to a shared repository.** Amending changes the commit hash, which will cause conflicts for other team members who have already pulled the original commit.

## Interview Questions
1.  **What happens to the original commit after you run `git commit --amend`?**
    The original commit becomes "orphaned" (not reachable from any branch tip) and will eventually be cleaned up by Git's garbage collection (GC).
2.  **How do you add a file to the last commit without changing the message?**
    `git add <file>` followed by `git commit --amend --no-edit`.
3.  **Why is it dangerous to amend pushed commits?**
    It rewrites history. Anyone else who pulled the original commit will have a divergent history, requiring complex manual merges or force pulls.

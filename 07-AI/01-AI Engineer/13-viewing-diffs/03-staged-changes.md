# Viewing Staged Changes

## Summary
Viewing staged changes (`git diff --staged` or `--cached`) allows you to review what exactly will be included in the next commit. This is a critical "sanity check" step in the AI development lifecycle to avoid committing temporary debug code, large data files, or incorrect configuration values.

## Detailed Explanation
The "staging area" (or index) sits between your working directory and the commit history.

### The Command
- **View what is ready to be committed:**
  ```bash
  git diff --staged
  ```
  (Note: `git diff --cached` is an alias for the same command).

### Why it matters for AI Engineers
1.  **Selective Committing:** You might have modified a training script and also added some `print` statements for debugging. By using `git add -p` (patch mode) to stage only the training script changes, `git diff --staged` helps you verify that the `print` statements are NOT going into the commit.
2.  **Config Integrity:** AI projects often have complex YAML/JSON configs. Staging a config change and diffing it ensures you didn't accidentally change a learning rate from `1e-4` to `1e-1`.
3.  **Preventing Data Leaks:** If you accidentally `git add .`, checking staged changes can help you spot large `.csv` or `.pt` files that shouldn't be in the repo before you finalize the commit.

### Workflow Example
1.  Modify `train.py`.
2.  `git add train.py`.
3.  `git diff --staged` -> "Ah, I left a `break` in the loop for testing. Let me fix that."

## Interview Questions
1.  **What is the difference between `git diff` and `git diff --staged`?**
    `git diff` shows changes in the working directory that are NOT yet staged. `git diff --staged` shows changes that ARE staged and ready for the next commit.
2.  **If you've already run `git add file.py`, how do you see the changes you just added?**
    `git diff --staged -- file.py`
3.  **How do you unstage a file if you see something wrong in the staged diff?**
    `git restore --staged <file>` or `git reset HEAD <file>`.

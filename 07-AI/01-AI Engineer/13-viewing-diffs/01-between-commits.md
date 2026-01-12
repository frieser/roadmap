# Viewing Diffs Between Commits

## Summary
The `git diff` command is used to compare different versions of the codebase across various commits. This is essential for understanding what changed over time, tracking down regressions, or reviewing implementation details of specific features. For AI Engineers, this often involves comparing changes in model architectures, hyperparameter settings, or data preprocessing logic.

## Detailed Explanation
Comparing commits allows you to see the exact lines of code that were added, modified, or removed between two points in history.

### Basic Syntax
- **Compare two specific commits:**
  ```bash
  git diff <commit-hash-1> <commit-hash-2>
  ```
- **Compare a commit with its parent:**
  ```bash
  git show <commit-hash>
  ```
- **Compare the current state with a specific commit:**
  ```bash
  git diff <commit-hash> HEAD
  ```

### AI Engineering Use Cases
1.  **Regression Analysis:** If a model's performance suddenly drops, you can compare the current commit with the last known "good" commit to identify changes in the training loop or data augmentation.
2.  **Hyperparameter Tracking:** If you haven't been using a dedicated experiment tracker (like W&B or MLflow) consistently, comparing commits where `config.yaml` or `params.py` was modified helps reconstruct experiment settings.
3.  **Code Review for Research:** When reviewing a colleague's research implementation, diffing their feature branch commits against the base helps focus on the mathematical changes or architectural shifts.

### Useful Options
- `--stat`: Shows a summary of changes (files changed, insertions, deletions).
- `--color-words`: Shows a word-level diff, which is useful for small changes in mathematical formulas or configuration strings.
- `-w`: Ignores whitespace changes, helpful when reformatting Python code (PEP 8).

## Interview Questions
1.  **How do you see the changes made in the last 3 commits for a specific file?**
    `git diff HEAD~3 HEAD -- path/to/file`
2.  **What is the difference between `git diff A B` and `git diff A...B`?**
    `git diff A B` compares the tips of both commits directly. `git diff A...B` compares the common ancestor of A and B with the tip of B (essentially showing what changed in B since it diverged from A).
3.  **In an ML project, why might `git diff` be insufficient for comparing model versions, and what are the alternatives?**
    `git diff` works on text files. Models are often binary blobs (e.g., `.pth`, `.h5`). Alternatives include DVC (Data Version Control) for binary diffing or specialized experiment trackers for metadata.

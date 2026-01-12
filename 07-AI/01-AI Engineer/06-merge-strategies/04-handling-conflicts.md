---
tags: ['ai', 'roadmap', 'git']
---

## Summary
A merge conflict occurs when Git is unable to automatically reconcile differences between two versions of the same file. For AI Engineers, this frequently happens when two people modify the same hyperparameter config, update the same line in a model architecture, or change a shared data preprocessing script. Resolving conflicts is a manual but essential process to ensure project integrity.

## Detailed Explanation

### How Conflicts Happen
If `Branch A` changes line 10 of `train.py` to `lr=0.01` and `Branch B` changes it to `lr=0.05`, Git doesn't know which one you want. It pauses the merge and asks for help.

### The Conflict Anatomy
When you open a conflicted file, you'll see these markers:
```python
<<<<<<< HEAD
learning_rate = 0.01  # Your current branch's version
=======
learning_rate = 0.05  # The version from the branch you are merging
>>>>>>> feature-branch
```

### The Resolution Workflow
1.  **Identify**: Run `git status` to see which files are "Unmerged."
2.  **Open**: Use a text editor (VS Code's "Merge Editor" is excellent for this).
3.  **Decide**: Remove the markers (`<<<<`, `====`, `>>>>`) and keep the correct code (or a combination of both).
4.  **Stage**: `git add <file_name>` to tell Git the conflict is resolved.
5.  **Commit**: `git commit` to finalize the merge.

### AI Context: Common Conflict Scenarios
*   **Requirements/Environment**: Two people adding different libraries to `requirements.txt`.
*   **Notebooks (`.ipynb`)**: Conflicts in Jupyter Notebooks are notoriously difficult because they are JSON files.
    *   *Tip*: Use tools like **nbdime** or **Jupytext** to make notebook conflicts easier to manage.
*   **Config Files**: Collisions in `.yaml` or `.json` experiment configs.

### Avoiding Conflicts
*   **Communicate**: Tell your team if you're making major changes to a shared file.
*   **Small PRs**: Smaller changes are less likely to conflict.
*   **Frequent Syncing**: Regularly pull/rebase from `main` to catch conflicts early.

## Interview Questions

**Q: What is a merge conflict in Git?**
**A:** A merge conflict occurs when Git cannot automatically merge changes from two branches because they have conflicting modifications to the same part of the same file. Git requires manual intervention to decide which changes to keep.

**Q: How do you identify which files have conflicts during a merge?**
**A:** You can run `git status`. Conflicted files will be listed under the "Unmerged paths" section, typically marked as "both modified."

**Q: What are the `<<<<<<<`, `=======`, and `>>>>>>>` markers?**
**A:** These are conflict markers. The code between `<<<<<<<` and `=======` is from your current branch (HEAD). The code between `=======` and `>>>>>>>` is from the branch being merged in. You must remove these markers as part of the resolution process.

**Q: How do you abort a merge if you realize the conflicts are too complex to handle right now?**
**A:** Run `git merge --abort`. This will stop the merge process and restore your repository to its state before the merge attempt began.

# Viewing Unstaged Changes

## Summary
Viewing unstaged changes is the process of comparing your current working directory with the state of the staging area (the index). It answers the question: "What have I changed since I last staged files or since the last commit?" For AI Engineers, this is the most common way to track local experimentation.

## Detailed Explanation
By default, `git diff` shows unstaged changes.

### The Command
- **Show all local modifications not yet added to the index:**
  ```bash
  git diff
  ```
- **Show changes for a specific file or directory:**
  ```bash
  git diff path/to/model.py
  ```

### AI Engineering Context
1.  **Iterative Refinement:** While tuning a neural network, you might make dozens of small changes to the architecture. `git diff` helps you keep track of these "live" changes before you decide they are worth "keeping" (staging).
2.  **Debugging Data Loaders:** If your data pipeline starts behaving strangely, `git diff` shows exactly what you tweaked in the `Dataset` class since the last stable state.
3.  **Notebook Comparisons:** Standard `git diff` is notoriously bad for Jupyter Notebooks (`.ipynb`). AI Engineers often use tools like `nbdime` to get meaningful diffs of unstaged notebook changes.

### Tips for Better Diffs
- **Diffing by word:** `git diff --word-diff` is great for seeing changes in long mathematical expressions or list of layers.
- **Limiting output:** Use `git diff -- . ':!notebooks/*'` to see changes excluding the notebooks directory.

## Interview Questions
1.  **Why does `git diff` sometimes return no output even though you know you changed a file?**
    Because the file has already been staged (`git add`). Use `git diff --staged` instead.
2.  **How can you see unstaged changes while ignoring all changes that only involve whitespace?**
    `git diff -w` or `git diff --ignore-all-space`.
3.  **In a team setting, why should you check your unstaged changes before running `git add .`?**
    To ensure you aren't accidentally including "scratchpad" code, sensitive API keys in scripts, or local-only path changes that would break the pipeline for others.

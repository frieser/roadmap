# Viewing Diffs Between Branches

## Summary
Comparing branches is a core Git workflow for reviewing features before merging. It helps AI Engineers understand the impact of experimental changes (like a new loss function or transformer block) relative to the stable production or main branch.

## Detailed Explanation
Branch comparison highlights the evolution of a feature or experiment.

### Basic Commands
- **Compare the tips of two branches:**
  ```bash
  git diff branch1 branch2
  ```
- **Compare the tip of a feature branch with the point it diverged from main:**
  ```bash
  git diff main...feature-branch
  ```
- **List files that differ between branches:**
  ```bash
  git diff --name-only main feature-branch
  ```

### AI Engineering Applications
1.  **A/B Experimentation:** When testing a new data preprocessing pipeline in a separate branch, a branch diff ensures no unintended side effects were introduced to the baseline logic.
2.  **Collaborative Research:** If multiple researchers are working on different optimization techniques in separate branches, diffing branches helps in merging efforts or identifying conflicting approaches.
3.  **Deployment Verification:** Comparing a `staging` branch with `production` to ensure all model updates and API changes are ready for release.

### Advanced Comparisons
- **Visual Diff Tools:** For complex changes in ML pipelines, using visual tools like `meld`, `vscode` diff viewer, or GitHub's PR interface is often preferred over CLI output.
- **Diffing specific directories:**
  ```bash
  git diff main:src/models feature-branch:src/models
  ```

## Interview Questions
1.  **How do you check which files have changed in your current branch compared to the `main` branch?**
    `git diff --name-only main`
2.  **What does the triple-dot syntax (`git diff main...feature`) do?**
    It shows the changes in the `feature` branch since it started from `main`. It's equivalent to `git diff $(git merge-base main feature) feature`.
3.  **How would you use Git to compare the performance metrics recorded in two different branches?**
    If metrics are stored in a text file (e.g., `results.json`), you can run `git diff branch1 branch2 -- results.json`. If using DVC, you would use `dvc metrics diff`.

---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Squashing is the process of condensing multiple commits into a single commit. This is typically done during an interactive rebase or when merging a feature branch into the main branch. For AI Engineers, squashing is invaluable for cleaning up "noisy" histories filled with commits like `fixed typo`, `debug print`, or `trying again`. It ensures that the final project history only contains significant, working milestones.

## Detailed Explanation

### Why Squash?
During AI development, you might commit every time you fix a small error in your training loop.
*   **Before Squash**: `Init train`, `fix bug`, `fix typo`, `add log`, `final train`.
*   **After Squash**: `Feature: Implement optimized training loop with logging`.

### How to Squash

#### 1. Interactive Rebase
```bash
git rebase -i HEAD~5
```
In the editor, change `pick` to `squash` (or `s`) for all commits except the first one. Git will then combine them and ask you for a new, unified commit message.

#### 2. Squash Merge (GitHub/CLI)
When merging a branch, you can squash all its commits into one:
```bash
# On main branch
git merge --squash feature-branch
git commit -m "Implement new feature (squashed)"
```
GitHub also provides a "Squash and merge" button in Pull Requests.

### Benefits for AI Teams
*   **Linear History**: Combined with a rebase-first workflow, it keeps the history a straight line.
*   **Easier Reverts**: If a feature is buggy, you only have to revert one commit, not five.
*   **Better Code Reviews**: Reviewers see one clean set of changes rather than a chaotic timeline of your development process.

### Drawbacks
*   **Loss of Detail**: You lose the granular history of how you arrived at the final solution. If those intermediate "failed" attempts were actually useful, squashing will hide them.

## Interview Questions

**Q: What does it mean to "squash" commits in Git?**
**A:** Squashing means taking multiple consecutive commits and combining them into a single, new commit with a unified message. It is used to clean up a messy commit history before merging a feature into the main codebase.

**Q: What is the difference between a standard merge and a "squash merge"?**
**A:** A standard merge creates a merge commit that joins two histories, keeping all individual commits from the feature branch visible. A squash merge takes all the changes from the feature branch, applies them as a single new commit on the target branch, and does not preserve the feature branch's individual history.

**Q: When should you use `fixup` instead of `squash` during an interactive rebase?**
**A:** Both combine the commit with the previous one. However, `squash` prompts you to edit and combine the commit messages, whereas `fixup` discards the current commit message and keeps the message of the previous commit. `fixup` is better for small corrections where the original message is already sufficient.

**Q: Why is squashing useful in a Pull Request workflow?**
**A:** It allows a developer to iterate quickly and commit frequently during the development phase. Once the PR is approved, squashing creates a single, high-quality commit on the main branch, which makes the repository history much easier to read and maintain for the entire team.

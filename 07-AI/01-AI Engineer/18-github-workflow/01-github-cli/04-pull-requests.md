## Summary
Pull Request (PR) management is the core of collaborative development. `gh pr` allows AI Engineers to create, review, and merge PRs directly, streamlining the peer-review process for model changes and code optimizations.

## Detailed Explanation
### **PR Life Cycle**
1. **Creation**: 
   ```bash
   gh pr create --title "Update training hyperparameters" --body "Optimized learning rate for better convergence."
   ```
2. **Reviewing**: 
   - `gh pr list` to see active PRs.
   - `gh pr checkout <number>` to test the code locally.
   - `gh pr diff` to see changes.
3. **Approval/Feedback**: 
   - `gh pr review <number> --approve`
   - `gh pr review <number> --comment -b "Check the loss curve."`
4. **Merging**: 
   ```bash
   gh pr merge <number> --merge --delete-branch
   ```

### **AI Context: Reviewing Large Diffs**
AI code often involves large config files or data schema changes. `gh pr view --web` can be used when the terminal output is too dense for visual inspection of complex changes.

### **Checks and Status**
Before merging, it is crucial to check if CI/CD tests (like model unit tests) passed:
```bash
gh pr checks <number>
```

## Interview Questions
- **Q: How do you check out a pull request locally using GitHub CLI?**
- **A:** `gh pr checkout <number>`.

- **Q: What command would you use to see the status of CI checks for a PR?**
- **A:** `gh pr checks`.

- **Q: How do you create a draft pull request?**
- **A:** `gh pr create --draft`.

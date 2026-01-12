## Summary
Issue management via the GitHub CLI (`gh issue`) enables efficient tracking of bugs, features, and tasks. For AI Engineers, this is vital for tracking model performance regressions or data pipeline failures without leaving the terminal environment.

## Detailed Explanation
### **Workflow with `gh issue`**
- **Listing Issues**: `gh issue list` (Defaults to open issues).
- **Creating an Issue**: 
  ```bash
  gh issue create --title "Fix data loader bottleneck" --body "The current data loader is too slow for GPU utilization."
  ```
- **Viewing an Issue**: `gh issue view <number>` (Displays description and comments).
- **Closing/Reopening**: `gh issue close <number>` or `gh issue reopen <number>`.

### **Filtering and Searching**
AI projects can have hundreds of issues. Filtering helps:
- `gh issue list --label "bug" --assignee "@me"`
- `gh issue list --search "tensor mismatch"`

### **Developing from an Issue**
The `gh issue develop` command is a powerful feature that creates a branch linked to the issue:
```bash
gh issue develop 42 --name "fix-bottleneck"
```

## Interview Questions
- **Q: How do you list only the issues assigned to you using `gh`?**
- **A:** `gh issue list --assignee "@me"`.

- **Q: What command creates a new branch and links it to a specific issue?**
- **A:** `gh issue develop <issue-number>`.

- **Q: How can you add a comment to an issue from the CLI?**
- **A:** Use `gh issue comment <number> --body "Message"`.

# Issue Management with CLI

## Summary
`gh issue` commands allow you to list, view, create, and close issues from the command line.

## Detailed Explanation

### Listing
```bash
gh issue list --assignee "@me"
```

### Creating
```bash
gh issue create --title "Bug in login" --body "Steps to reproduce..."
```
Or interactive mode: `gh issue create`.

### Viewing
```bash
gh issue view 123
```
Displays the issue content and comments in the terminal.

## Interview Questions
**Q: Can you filter issues by label?**
**A:** Yes, `gh issue list --label "bug"`.

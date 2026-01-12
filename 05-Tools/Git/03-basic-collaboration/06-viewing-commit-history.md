# Viewing Commit History

## Summary
`git log` is the primary tool to view the project's history. It allows you to filter, search, and visualize the timeline of commits, helping you understand how the project evolved.

## Detailed Explanation

### Common Options
*   `git log`: Standard list.
*   `git log --oneline`: Compact, one commit per line.
*   `git log --graph`: ASCII graph showing branching and merging.
*   `git log -p`: Show the patch (diff) introduced in each commit.
*   `git log --author="Name"`: Filter by author.
*   `git log --grep="keyword"`: Search commit messages.

### Visualizing
```bash
# The "Super Log" alias many developers use
git log --oneline --graph --decorate --all
```

### Go-specific Context
When auditing a Go project for security, you might check changes to `go.mod`:
```bash
git log -p go.mod
```
This shows the history of dependency updates, allowing you to see when a specific library version was upgraded or downgraded.

## Interview Questions
**Q: How do you see the history of a specific file?**
**A:** `git log <filename>`. To see the diffs for that file, use `git log -p <filename>`.

**Q: How can you limit `git log` to the last 3 commits?**
**A:** `git log -n 3` or `git log -3`.

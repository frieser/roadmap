# Issues

## Summary
GitHub Issues are used to track ideas, feedback, tasks, or bugs for work on GitHub. They are the central communication channel for planning work before writing code.

## Detailed Explanation

### Features
*   **Assignees**: Who is responsible.
*   **Labels**: Categorization (bug, enhancement, documentation).
*   **Milestones**: Grouping issues into a release target (v1.0).
*   **Linking**: Reference issues in other issues or PRs (`#123`).

### Closing Issues
You can close issues manually or automatically via commit messages:
```bash
git commit -m "fix login bug, closes #42"
```
When this commit is merged into the default branch, issue #42 closes automatically.

### Go-specific Context
The Go project uses GitHub Issues extensively. Because Go is stable, the issue tracker is strictly triaged. A "Proposal" process is used for language changes, starting as an Issue labeled `Proposal`.

## Interview Questions
**Q: Can you convert an Issue to a Pull Request?**
**A:** Not directly, but you can create a branch linked to the issue, or use the "Create branch" button in the Issue sidebar.

**Q: What is a pinned issue?**
**A:** An issue that maintainers stick to the top of the list to increase visibility (e.g., FAQ or roadmap).

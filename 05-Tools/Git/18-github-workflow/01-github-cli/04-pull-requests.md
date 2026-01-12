# Pull Requests with CLI

## Summary
`gh pr` is arguably the most useful part of the CLI. It streamlines the creation and checkout of Pull Requests.

## Detailed Explanation

### Creating
```bash
gh pr create
```
This pushes your current branch (if needed), prompts for title/body (or opens your editor), and creates the PR.

### Checking Out
```bash
gh pr checkout 123
```
This fetches the PR branch and switches to it, even if it's from a fork you haven't added as a remote. This is a massive time saver for maintainers.

### Merging
```bash
gh pr merge 123 --squash --delete-branch
```

## Interview Questions
**Q: How do you see the diff of a PR?**
**A:** `gh pr diff 123`.

**Q: How do you approve a PR?**
**A:** `gh pr review 123 --approve`.

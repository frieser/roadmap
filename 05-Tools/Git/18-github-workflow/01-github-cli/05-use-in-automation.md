# Automation with GitHub CLI

## Summary
The GitHub CLI is designed for scripting. You can output data in JSON format and use `jq` to parse it, enabling powerful automation scripts.

## Detailed Explanation

### Usage
```bash
gh issue list --json title,number,labels | jq '.[].title'
```

### GitHub Actions
`gh` is pre-installed in GitHub Actions runners. You can use it in workflows to create comments, close issues, or trigger other workflows.
```yaml
- run: gh issue close ${{ github.event.issue.number }}
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Go-specific Context
You can write a Go program that calls `gh` as a subprocess to build custom dev tools for your team (e.g., a tool that generates a weekly report of merged PRs).

## Interview Questions
**Q: Why use `gh` in scripts instead of `curl` with the API?**
**A:** `gh` handles authentication automatically, has simpler syntax, and is maintained by GitHub.

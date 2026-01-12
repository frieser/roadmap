# Workflow Status

## Summary
You can check the status of a workflow programmatically or visually. Badges are images that show the passing/failing status of your `main` branch.

## Detailed Explanation

### Badges
You can copy the Markdown for a status badge from the Actions tab and paste it into your `README.md`.
`![CI](https://github.com/user/repo/actions/workflows/ci.yml/badge.svg)`

### Job Status Check
In a workflow, you can run steps based on the success/failure of previous steps.
```yaml
- name: Notify Slack on Failure
  if: failure()
  uses: slackapi/slack-github-action@v1.23.0
```

### Go-specific Context
In Go projects, "Green Build" (passing status) is often a requirement for merging PRs (Branch Protection Rule).

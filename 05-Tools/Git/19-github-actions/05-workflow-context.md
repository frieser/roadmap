# Workflow Context

## Summary
The `context` contains information about the workflow run, such as the branch name, commit SHA, actor (user), and event details. You access it using `${{ }}` syntax.

## Detailed Explanation

### Common Contexts
*   **`github.sha`**: The commit hash that triggered the run.
*   **`github.ref`**: The branch or tag ref (e.g., `refs/heads/main`).
*   **`github.actor`**: The username of the person who triggered the run.
*   **`github.event_name`**: The name of the event (e.g., `push`).

### Conditionals
```yaml
steps:
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: ./deploy.sh
```

### Go-specific Context
When versioning a build, you often pass the SHA to the linker:
```yaml
- run: go build -ldflags "-X main.version=${{ github.sha }}"
```

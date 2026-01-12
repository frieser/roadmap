# Secrets and Env Vars

## Summary
Secrets allow you to store sensitive information (API keys, passwords) securely. Environment variables allow you to configure build parameters.

## Detailed Explanation

### Secrets
*   **Storage**: Settings > Secrets and variables > Actions.
*   **Usage**: `${{ secrets.MY_API_KEY }}`.
*   **Security**: Masked in logs (displayed as `***`).

### Environment Variables
*   **Scope**: Workflow level, Job level, or Step level.
*   **Usage**: `env.MY_VAR` or `$MY_VAR` in shell.

```yaml
env:
  GO_VERSION: '1.21'

steps:
  - run: echo "Deploying with key ${{ secrets.AWS_KEY }}"
```

### Go-specific Context
If your Go tests require a database password, store it as a Secret and pass it as an env var:
```yaml
- run: go test ./...
  env:
    DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

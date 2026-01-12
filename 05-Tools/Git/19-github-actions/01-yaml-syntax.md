# YAML Syntax for GitHub Actions

## Summary
GitHub Actions workflows are defined in YAML files located in `.github/workflows/`. They consist of triggers, jobs, and steps.

## Detailed Explanation

### Basic Structure
```yaml
name: CI
on: [push]  # Trigger
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4  # Action
      - name: Run script
        run: echo "Hello World"    # Shell command
```

### Key Keywords
*   `on`: Events that trigger the workflow.
*   `jobs`: Parallel units of work.
*   `steps`: Sequential tasks within a job.
*   `uses`: Reusing a community action.
*   `run`: Executing a shell command.

### Go-specific Context
A standard Go workflow:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-go@v5
    with:
      go-version: '1.21'
  - run: go test ./...
```

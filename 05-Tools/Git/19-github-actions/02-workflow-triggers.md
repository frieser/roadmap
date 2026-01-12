# Workflow Triggers

## Summary
Triggers define *when* a workflow should run. You can trigger on Git events (push, PR), scheduled times, or manual inputs.

## Detailed Explanation

### Common Triggers
*   **push**: Runs on every push to specific branches.
    ```yaml
    on:
      push:
        branches: [ "main" ]
    ```
*   **pull_request**: Runs when a PR is opened or updated.
    ```yaml
    on:
      pull_request:
        branches: [ "main" ]
    ```
*   **workflow_dispatch**: Adds a "Run workflow" button in the UI for manual triggering.
*   **release**: When a release is published.

### Go-specific Context
For Go projects, you typically want to run tests on `push` to any branch (to catch errors early) and strictly on `pull_request` to `main` (to block bad merges).

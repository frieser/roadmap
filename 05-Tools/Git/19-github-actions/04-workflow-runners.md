# Workflow Runners

## Summary
A runner is the server that executes your workflow. GitHub provides hosted runners (Ubuntu, Windows, macOS), or you can host your own.

## Detailed Explanation

### GitHub-hosted Runners
*   **`ubuntu-latest`**: Most common, cheapest (in terms of billable minutes), fastest startup.
*   **`windows-latest`**: Necessary for .NET or Windows-specific testing.
*   **`macos-latest`**: Necessary for iOS/macOS builds. Expensive (10x billing multiplier).

### Self-hosted Runners
You can install the runner agent on your own AWS EC2 instance or on-prem server.
*   **Pros**: Free (no minute limits), access to internal network resources.
*   **Cons**: You have to maintain the server.

### Go-specific Context
Go is cross-platform. You can cross-compile for Windows/Mac from Linux (`GOOS=windows go build`), so you usually only need `ubuntu-latest` runners, saving money.

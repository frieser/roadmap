# GitHub Security

## Summary
GitHub has a suite of security features to protect your code supply chain.

## Detailed Explanation

### Dependabot
*   **Alerts**: Notifies you of vulnerable dependencies.
*   **Updates**: Automatically opens PRs to upgrade libraries.

### Secret Scanning
*   Scans commits for known patterns (AWS keys, Stripe tokens).
*   If found, it notifies you (and sometimes the provider to revoke the token).

### CodeQL (Advanced Security)
*   Semantic code analysis engine.
*   Finds bugs like SQL injection or Cross-site Scripting (XSS) by analyzing data flow.

### Go-specific Context
Dependabot fully supports `go.mod`. CodeQL has deep support for Go, detecting common concurrency bugs or unsafe pointer usage.

## Interview Questions
**Q: What is a CODEOWNERS file?**
**A:** A file that defines who is responsible for specific paths in the repo. GitHub automatically requests their review when those files change.

# Git Hooks Use Cases

## Summary
Hooks automate the development workflow, ensuring quality and consistency before code even reaches the shared repository.

## Detailed Explanation

### Common Scenarios
1.  **Code Quality**: Run linters (`golangci-lint`) before commit.
2.  **Security**: Scan for AWS keys or secrets in code before commit.
3.  **Convention**: Ensure commit message starts with a ticket number.
4.  **Notification**: Send an email/webhook when a push is received (Server-side).

### Go-specific Context
In Go projects, a `pre-commit` hook is often used to run:
*   `go fmt` (formatting)
*   `go mod tidy` (dependency cleanup)
*   `staticcheck` (advanced linting)

## Interview Questions
**Q: How do you bypass a pre-commit hook?**
**A:** `git commit --no-verify` (or `-n`). Use this sparingly!

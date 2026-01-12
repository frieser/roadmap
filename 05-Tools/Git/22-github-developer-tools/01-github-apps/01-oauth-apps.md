# OAuth Apps vs GitHub Apps

## Summary
GitHub offers two ways to integrate: **OAuth Apps** (acting as the user) and **GitHub Apps** (acting as an independent bot/actor).

## Detailed Explanation

### OAuth Apps
*   **Identity**: "Login with GitHub".
*   **Permissions**: Broad scopes (e.g., `repo` gives access to ALL private repos).
*   **Actor**: Acts *as* the user.

### GitHub Apps (Recommended)
*   **Identity**: Independent application (e.g., "Dependabot").
*   **Permissions**: Granular (e.g., "Read access to Issues only").
*   **Installation**: Installed on specific repositories.
*   **Actor**: Acts as itself (bot).

### Go-specific Context
If you are building a tool for your team, build a GitHub App. It is more secure and doesn't require sharing user passwords/tokens.

## Interview Questions
**Q: Which one supports Webhooks?**
**A:** Both, but GitHub Apps have first-class support for granular webhook events.

# Client vs Server Hooks

## Summary
Git hooks are scripts that run automatically every time a particular event occurs in a Git repository. They are divided into two groups: Client-side (triggered by operations like committing/merging) and Server-side (triggered by network operations like receiving pushed commits).

## Detailed Explanation

### Client-side
*   **Where**: Local `.git/hooks/`.
*   **Triggers**: `git commit`, `git checkout`, `git merge`.
*   **Use**: Linting, formatting, message checking.
*   **Note**: Not shared when cloning (security feature). You must use a tool (like `pre-commit`) to sync them.

### Server-side
*   **Where**: The remote repository server.
*   **Triggers**: `git push` (on the receiving end).
*   **Use**: Enforcing policy (rejecting commits without specific format), triggering CI/CD, notifying Slack.

## Interview Questions
**Q: Can I enforce a client-side hook on my team?**
**A:** Not strictly via Git itself, as users can bypass them (`--no-verify`). However, you can use server-side hooks to reject code that clearly didn't pass the client checks.

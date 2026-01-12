# Pre-Push Hook

## Summary
The `pre-push` hook runs during `git push`, after the remote refs have been determined but before any objects are transferred. It is the last line of defense before sharing code.

## Detailed Explanation

### Use Cases
*   **Run Tests**: Run the full test suite (`go test ./...`) to ensure you aren't breaking the build.
*   **Linting**: Run expensive linters.
*   **Branch Protection**: Prevent pushing to `main` directly (if server-side protection isn't enough).

### Pros & Cons
*   **Pros**: Keeps the remote clean.
*   **Cons**: Can make `git push` slow.

## Interview Questions
**Q: If pre-push fails, what happens?**
**A:** The push is aborted. Nothing is sent to the remote.

**Q: Can I push anyway?**
**A:** Yes, `git push --no-verify`.

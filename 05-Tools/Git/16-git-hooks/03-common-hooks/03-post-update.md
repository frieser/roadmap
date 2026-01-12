# Post-Update Hook

## Summary
The `post-update` hook is a server-side hook. It runs after the remote repository has been updated (after the push is accepted).

## Detailed Explanation

### Use Cases
*   **Notifications**: Trigger a CI build (Jenkins/GitHub Actions usually use webhooks, but this is the Git-native way).
*   **Mirroring**: Push the changes to a backup server.
*   **Website Deploy**: If pushing to a web server, checkout the files to the `/var/www` directory.

### Go-specific Context
In a private Go module proxy server, a `post-update` hook might trigger the proxy to refresh its cache of the module.

## Interview Questions
**Q: Can `post-update` reject the push?**
**A:** No, the update has already happened. Use `pre-receive` or `update` hooks to reject pushes.

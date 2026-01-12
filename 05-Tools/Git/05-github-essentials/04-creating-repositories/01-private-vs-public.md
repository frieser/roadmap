# Private vs Public Repositories

## Summary
Repositories on GitHub can be Public (visible to everyone) or Private (visible only to you and invited collaborators).

## Detailed Explanation

### Public Repositories
*   **Visibility**: Anyone on the internet can see the code.
*   **Cost**: Free.
*   **Features**: Free GitHub Actions (unlimited for public), Pages, Wiki.
*   **Use Case**: Open Source projects, portfolios, libraries.

### Private Repositories
*   **Visibility**: Restricted to specific users.
*   **Cost**: Free (with limits on collaborators/minutes) or Paid (Pro/Team).
*   **Features**: Limited Action minutes on free tier.
*   **Use Case**: Proprietary software, coursework, personal experiments.

### Go-specific Context
*   **Go Modules**: `go get` works seamlessly with Public repos.
*   **Private Modules**: To use a Private repo as a Go module dependency, you must set `GOPRIVATE=github.com/my-org/*` in your environment. This tells the Go toolchain to bypass the public proxy (which can't see your private code) and fetch directly from GitHub (requiring authentication).

## Interview Questions
**Q: Can you convert a public repo to private?**
**A:** Yes, in the Settings tab "Danger Zone". However, you lose stars/watchers, and open forks will be detached.

**Q: Who owns the code in a public repository?**
**A:** The author retains copyright, but by making it public on GitHub, you grant others the right to view and fork it (GitHub ToS). You should add a LICENSE (like MIT or Apache) to clarify usage rights.

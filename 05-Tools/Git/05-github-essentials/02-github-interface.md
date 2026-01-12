# GitHub Interface

## Summary
The GitHub web interface provides a visual way to interact with Git repositories. It adds features on top of Git, like Issues, Pull Requests, Actions, and Projects.

## Detailed Explanation

### Key Views
1.  **Dashboard**: Your homepage. Shows activity from people you follow and repositories you star.
2.  **Repository View**:
    *   **Code**: Browse files and README.
    *   **Issues**: Bug tracker and feature requests.
    *   **Pull Requests**: Code review interface.
    *   **Actions**: CI/CD pipelines.
    *   **Security**: Vulnerability alerts (Dependabot).
    *   **Insights**: Graphs of contributions, traffic, and forks.
    *   **Settings**: Repo-level configuration (collaborators, branches, secrets).

### Go-specific Context
*   **Go Reference Link**: GitHub automatically detects `go.mod` files and links to pkg.go.dev documentation for your project.
*   **Releases**: When you create a Release in the interface (tagging a version like `v1.0.0`), GitHub zips the source code. Go modules proxy uses these tags to serve specific versions to users.

## Interview Questions
**Q: Where do you find the clone URL in the interface?**
**A:** Under the green "Code" button on the main repository page.

**Q: What is the difference between "Watching", "Starring", and "Forking"?**
**A:**
*   **Watch**: Get notifications for activity.
*   **Star**: Bookmark (like).
*   **Fork**: Create your own copy of the repo to contribute back.

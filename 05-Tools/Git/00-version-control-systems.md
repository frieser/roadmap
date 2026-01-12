# Git (Version Control Systems)

## Summary
Git is a distributed version control system (VCS) that tracks changes in source code during software development. It allows multiple developers to work on the same project simultaneously without overwriting each other's work. By maintaining a complete history of changes, Git enables teams to audit code, roll back to previous versions, and manage complex feature development through branching.

## Detailed Explanation
Version Control is the backbone of modern backend engineering. It provides a "safety net" for code and a platform for collaboration.

### Core Concepts
*   **Repository (Repo):** A database containing all the files and their revision history.
*   **Commit:** A snapshot of your project at a specific point in time. Each commit has a unique SHA-1 hash.
*   **Branching:** Independent lines of development. The `main` branch usually holds production-ready code, while feature branches are used for experimentation.
*   **Merging:** Combining changes from different branches.
*   **Distributed Architecture:** Unlike centralized VCS (like SVN), every developer has a full copy of the repository history on their local machine.

### Why It's Essential for Backend
1.  **Traceability:** Identifying which change introduced a bug in a complex microservice architecture.
2.  **Deployment Safety:** Tagging specific commits for production releases ensures consistency across environments.
3.  **Code Review:** Facilitates peer review of database migrations, API changes, and security fixes before they reach production.

## Go-specific Context
In the Go ecosystem, Git is deeply integrated into the toolchain through Go Modules.

*   **Dependency Management:** The `go get` command uses Git to download modules. It maps package paths (e.g., `github.com/user/repo`) directly to Git repositories.
*   **Version Tagging:** Go modules use Semantic Versioning (SemVer) tags (e.g., `v1.2.3`) in Git to manage dependencies.
*   **Private Repositories:** When working with private Git repos, Go developers often set the `GOPRIVATE` environment variable to prevent the Go proxy from trying to fetch internal code.

```go
// Example of how Go modules reference Git versions in go.mod
module my-backend-app

go 1.23

require (
    github.com/gin-gonic/gin v1.10.0 // Managed via Git tags
    github.com/google/uuid v1.6.0
)
```

## Interview Questions
**Q: What is the difference between `git fetch` and `git pull`?**
**A:** `git fetch` downloads the latest changes from the remote repository but does not merge them into your local branch. `git pull` is a combination of `git fetch` followed by `git merge`, updating your current local branch with the remote changes immediately.

**Q: Explain the Git "Staging Area" (Index).**
**A:** The staging area is an intermediate layer between the working directory and the repository. It allows you to selectively choose which changes to include in the next commit, enabling "atomic commits" where only related changes are grouped together.

**Q: How do you resolve a merge conflict in Git?**
**A:** To resolve a conflict, you must manually edit the files containing conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`), choose the correct code, remove the markers, stage the resolved files with `git add`, and then complete the merge with `git commit`.

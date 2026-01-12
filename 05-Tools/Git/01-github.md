# GitHub

## Summary
GitHub is a cloud-based hosting service for Git repositories. It enhances the core Git functionality with collaboration tools like Pull Requests, Issue Tracking, and GitHub Actions for CI/CD. It is the most popular platform for open-source and enterprise software development, providing a centralized hub for code sharing and project management.

## Detailed Explanation
GitHub transforms Git from a command-line tool into a social and collaborative ecosystem.

### Key Features
*   **Pull Requests (PRs):** A mechanism for proposing changes to a repository. PRs allow for code reviews, automated testing, and discussion before code is merged into the main branch.
*   **Issues:** A built-in bug tracker and project management tool.
*   **GitHub Actions:** A powerful CI/CD platform that allows you to automate workflows (build, test, deploy) directly from your repository.
*   **GitHub Packages:** A software package hosting service for Docker images, npm packages, and Go modules.

### CI/CD with GitHub Actions
Workflows are defined in YAML files inside the `.github/workflows` directory. They are triggered by events like `push`, `pull_request`, or on a schedule.

## Go-specific Context
GitHub is the primary home for the Go ecosystem. Most open-source Go libraries are hosted here.

*   **Continuous Integration:** Using GitHub Actions to run `go test` and `go build` on every PR.
*   **Security:** GitHub's `dependabot` automatically checks Go dependencies for vulnerabilities and opens PRs to update them.
*   **Release Automation:** Using tools like `goreleaser` within GitHub Actions to create GitHub Releases with compiled binaries for multiple architectures.

```yaml
# Example GitHub Action for a Go Project
name: Go CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.23'
      - name: Build
        run: go build -v ./...
      - name: Test
        run: go test -v ./...
```

## Interview Questions
**Q: What is the purpose of a Pull Request?**
**A:** A Pull Request is a request to merge code from one branch (usually a feature branch) into another (usually main). It provides a platform for code review, team discussion, and running automated CI checks to ensure code quality before integration.

**Q: What are GitHub Actions Secrets?**
**A:** Secrets are encrypted environment variables used in GitHub Actions to store sensitive information like API keys, database credentials, or deployment tokens. They are never exposed in logs and can only be accessed by the workflow runner.

**Q: How do you handle private Go modules hosted on GitHub?**
**A:** You need to configure Git to use an SSH key or a Personal Access Token (PAT) for authentication, and set the `GOPRIVATE` environment variable (e.g., `GOPRIVATE=github.com/my-org/*`) to tell the Go toolchain to bypass the public proxy and checksum database.

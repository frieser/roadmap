# PR from a Fork

## Summary
Submitting a Pull Request from a fork is the standard way to contribute to Open Source. You make changes in your copy of the repository and request the original maintainers to pull those changes in.

## Detailed Explanation

### Steps
1.  **Fork** the target repo.
2.  **Clone** your fork.
3.  **Branch**: Create a feature branch `feat/add-xyz`.
4.  **Commit & Push** to your fork.
5.  **Open PR**: Go to the original repo. GitHub will often detect your push and show a banner "Compare & pull request".
    *   **Base repository**: Original repo (`main`).
    *   **Head repository**: Your fork (`feat/add-xyz`).

### Go-specific Context
When contributing to Go projects, you often need to ensure your `go.mod` changes (if any) are valid. CI checks on the original repo will run `go test` and `go mod tidy` against your PR code to verify it doesn't break the build.

## Interview Questions
**Q: What happens if the original repo changes while my PR is open?**
**A:** Your PR might get conflicts. You need to pull changes from `upstream/main` into your local branch, resolve conflicts, and push again to update the PR.

**Q: Can maintainers edit my PR?**
**A:** Yes, if you leave the "Allow edits by maintainers" checkbox checked (default). This allows them to fix typos or rebase without asking you to do it.

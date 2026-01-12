# Contribution Guidelines

## Summary
`CONTRIBUTING.md` is a file in the repository root that explains to potential contributors how they should help with the project. GitHub displays a link to this file when a user opens a new Issue or PR.

## Detailed Explanation

### What to include
1.  **Code of Conduct**: Link to `CODE_OF_CONDUCT.md`.
2.  **Getting Started**: How to set up the dev environment.
3.  **Testing**: How to run the test suite.
4.  **Style Guide**: Coding standards (linting rules).
5.  **Submission Process**: PR template, signing commits (DCO/CLA).

### CLA vs DCO
*   **CLA (Contributor License Agreement)**: Legal agreement stating you own the code and grant rights to the project.
*   **DCO (Developer Certificate of Origin)**: Simpler mechanism where you sign-off commits (`git commit -s`) asserting you have the right to submit the code.

### Go-specific Context
Go projects usually point to `go fmt` and `go vet` as mandatory steps.
They might also specify version requirements (e.g., "We support the last 2 major Go versions").

## Interview Questions
**Q: Why is a CONTRIBUTING.md file important?**
**A:** It reduces friction for new contributors and saves maintainers time by answering common questions ("How do I run tests?") upfront.

**Q: What is `git commit -s`?**
**A:** It adds a `Signed-off-by` trailer to the commit message, often used for DCO compliance.

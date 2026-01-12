# PR Guidelines

## Summary
Pull Request guidelines ensure that code contributions are high-quality, reviewable, and follow the project's standards. Many projects use a `PULL_REQUEST_TEMPLATE.md` to enforce a structure.

## Detailed Explanation

### Common Elements
1.  **Summary**: What does this change?
2.  **Type of Change**: Bug fix, Feature, Breaking change?
3.  **Checklist**:
    *   Tests added/passed?
    *   Documentation updated?
    *   Linting passed?
4.  **Related Issue**: Links to `#123`.

### Best Practices
*   **Small PRs**: Easy to review.
*   **Self-Review**: Comment on your own PR to explain complex logic before asking others.
*   **Screenshots**: If it's a UI change.

### Go-specific Context
*   **Go formatting**: PRs that fail `gofmt` are usually rejected automatically by CI.
*   **API Compatibility**: Go promises compatibility (Go 1 compatibility guarantee). A PR that breaks existing Go APIs will be rejected or require a major version bump strategy.

## Interview Questions
**Q: Where should you put the PR template?**
**A:** In the root directory, `.github/` directory, or `docs/` directory named `PULL_REQUEST_TEMPLATE.md`.

**Q: Why is "Draft PR" useful?**
**A:** It signals that work is in progress. CI might run, but the PR cannot be merged until marked "Ready for review".

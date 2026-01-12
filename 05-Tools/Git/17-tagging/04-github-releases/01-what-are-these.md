# GitHub Releases

## Summary
GitHub Releases are a wrapper around Git tags. They turn a tag into a downloadable software product.

## Detailed Explanation

### Features
*   **Release Notes**: Automatically generated or manually written changelog.
*   **Assets**: Binary files (`.exe`, `.zip`) attached to the release.
*   **Pre-release**: Marking a tag as "Beta" so it doesn't show as "Latest".

### Go-specific Context
GoReleaser is a popular tool that automates this:
1.  Builds Go binaries for all platforms (Windows, Linux, Mac).
2.  Creates a Git tag.
3.  Uploads the binaries to GitHub Releases.
4.  Generates Homebrew formulas.

## Interview Questions
**Q: How does a Release relate to a Tag?**
**A:** A Release is metadata *associated* with a Tag. You cannot have a Release without a Tag.

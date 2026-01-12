# Diff Between Commits

## Summary
`git diff` allows you to see the exact changes (additions and deletions) between any two points in the repository's history.

## Detailed Explanation

### Usage
```bash
git diff <commit-sha-A> <commit-sha-B>
```
This shows what happened between A and B.

### Output Format
*   **- (Red)**: Lines removed.
*   **+ (Green)**: Lines added.
*   **@@ ... @@**: Header context (line numbers).

### Go-specific Context
When auditing a library update, you might diff the two versions (tags):
```bash
git diff v1.0.0 v1.0.1
```
This helps you verify that only bug fixes were included and no malicious code or breaking changes slipped in.

## Interview Questions
**Q: How do you see the diff of only one file between commits?**
**A:** `git diff <sha1> <sha2> -- path/to/file.go`.

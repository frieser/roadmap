# Commit Messages

## Summary
A commit message is a communication to your fellow developers (and your future self) about *why* a change was made. Good commit messages allow for automatic changelog generation and easier debugging.

## Detailed Explanation

### The Seven Rules
1.  Separate subject from body with a blank line.
2.  Limit the subject line to 50 characters.
3.  Capitalize the subject line.
4.  Do not end the subject line with a period.
5.  Use the imperative mood in the subject line (e.g., "Add feature" not "Added feature").
6.  Wrap the body at 72 characters.
7.  Use the body to explain **what** and **why** vs. **how**.

### Conventional Commits
A popular standard for structuring commit messages:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```
*   **fix**: patches a bug.
*   **feat**: introduces a new feature.
*   **chore**, **docs**, **style**, **refactor**, **perf**, **test**.

### Go-specific Context
The Go project has a strict commit message format:
```
archive/zip: fix panic in Reader.Open

The Reader.Open method could panic if...
Fixes #12345
```
It always starts with the package name (`archive/zip`).

## Interview Questions
**Q: Why use the imperative mood?**
**A:** It matches the way Git itself generates messages (e.g., "Merge branch 'feature'"). It completes the sentence "If applied, this commit will..."

**Q: What is `git commit --amend`?**
**A:** It allows you to modify the most recent commit message (and content) if you haven't pushed it yet.

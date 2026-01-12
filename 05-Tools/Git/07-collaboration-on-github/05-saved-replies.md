# Saved Replies

## Summary
Saved Replies are pre-written responses that you can insert into comments, issues, or pull requests. They save time for maintainers who frequently answer the same questions.

## Detailed Explanation

### How to use
1.  Go to Settings > Saved replies.
2.  Add a title (e.g., "Missing Reproduction").
3.  Add content: "Please provide a minimal reproduction case so we can debug this..."
4.  In a comment field, click the "Saved replies" icon (or type Ctrl+.) to insert it.

### Use Cases
*   Requesting more info.
*   Politely declining a feature.
*   Directing users to StackOverflow for support questions.
*   Welcome message for first-time contributors.

### Go-specific Context
Go maintainers might use saved replies to explain common "works as intended" behaviors in Go, such as loop variable capture semantics (before Go 1.22) or nil interface mechanics.

## Interview Questions
**Q: Are saved replies shared across the team?**
**A:** No, they are personal to your user account. However, you can document shared templates in the repo (like `CONTRIBUTING.md`).

# Automations

## Summary
GitHub Projects supports no-code automations to reduce manual overhead. These workflows run in response to changes in Issues or PRs.

## Detailed Explanation

### Examples
*   **Item Added**: When an issue is added to the project -> Set Status to "Todo".
*   **PR Merged**: When a linked PR is merged -> Move issue to "Done".
*   **PR Review Requested**: Move to "In Review".
*   **Auto-archive**: Archive items that have been "Done" for > 1 month.

## Interview Questions
**Q: Can I write custom scripts for project automation?**
**A:** Yes, via the GitHub GraphQL API, you can write external scripts (e.g., in Go) to manipulate project items programmatically.

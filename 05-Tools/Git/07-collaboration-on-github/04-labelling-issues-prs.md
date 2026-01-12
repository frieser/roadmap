# Labelling Issues and PRs

## Summary
Labels are colored tags used to categorize and filter Issues and Pull Requests. They help maintainers triage incoming work and help contributors find issues they can help with.

## Detailed Explanation

### Standard Labels
*   `bug`: Something isn't working.
*   `enhancement`: New feature.
*   `documentation`: Improvements to docs.
*   `duplicate`: This issue already exists.
*   `good first issue`: Good for newcomers.
*   `help wanted`: Extra attention needed.
*   `wontfix`: The team decided not to work on this.

### Usage
You can filter lists by clicking on labels. For example, search `is:issue is:open label:"good first issue" language:go`.

### Go-specific Context
The Go repo uses labels like `OS:Windows`, `Arch:ARM`, `net/http` to route issues to the specific team experts for that domain.

## Interview Questions
**Q: Can you create custom labels?**
**A:** Yes, in the repository issues label page. You can set the name, description, and color.

**Q: Can you apply labels automatically?**
**A:** Yes, using GitHub Actions (e.g., labeler action based on modified files) or Issue Templates.

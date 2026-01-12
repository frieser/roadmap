# GitHub Wikis

## Summary
Every GitHub repository comes with a Wiki section, which is actually a separate Git repository. It is useful for long-form documentation that doesn't fit in a README (e.g., architecture diagrams, comprehensive tutorials).

## Detailed Explanation

### Features
*   **Pages**: Create multi-page documentation.
*   **Sidebar**: Custom navigation.
*   **Editing**: Can be edited online or cloned locally.

### Cloning a Wiki
If your repo is `github.com/user/repo`, the wiki is `github.com/user/repo.wiki.git`.

### Go-specific Context
While Wikis are useful, the Go community often prefers placing documentation inside the code (for godoc) or in a `docs/` folder in the main repo, so that documentation versioning stays in sync with code versioning (which Wikis don't enforce strictly relative to the main repo tags).

## Interview Questions
**Q: Can a Wiki have restricted access different from the main repo?**
**A:** On public repos, Wikis are public. On private repos, they are private. You generally can't have a public wiki for a private repo on GitHub.

**Q: Does the Wiki support Markdown?**
**A:** Yes, it is the default format.

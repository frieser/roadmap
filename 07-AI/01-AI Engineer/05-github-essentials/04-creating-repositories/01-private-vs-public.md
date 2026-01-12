---
tags: ['ai', 'roadmap', 'github']
---

## Summary
When creating a repository on GitHub, you must choose between a Public or Private visibility setting. This decision is critical for AI Engineers, as it impacts collaboration, intellectual property protection, and portfolio building. Public repos are the foundation of open-source AI, while Private repos are essential for protecting proprietary models, datasets, and sensitive company code.

## Detailed Explanation

### 1. Public Repositories
*   **Visibility**: Anyone on the internet can see the code.
*   **Collaboration**: Anyone can fork the repo and propose changes via Pull Requests.
*   **AI Use Case**: Showcasing your "Side Projects," contributing to libraries like `scikit-learn`, or releasing open-weights models.
*   **Benefit**: Builds your reputation and allows for free community feedback.

### 2. Private Repositories
*   **Visibility**: Only you and explicitly invited collaborators can see the code.
*   **Collaboration**: Restricted to specific team members.
*   **AI Use Case**: Developing proprietary model architectures, handling sensitive training datasets, or working on confidential business logic.
*   **Benefit**: Complete control over who sees your research and intellectual property.

### Key Considerations for AI Projects

| Feature | Public | Private |
| :--- | :--- | :--- |
| **Secrets Management** | High Risk. One mistake leaks API keys to the world. | Lower Risk, but still requires good practices. |
| **Data Privacy** | NEVER put PII (Personally Identifiable Information) here. | Safer, but GitHub's Terms of Service still apply. |
| **GitHub Actions** | Free for public repos (unlimited minutes). | Limited free minutes (depending on plan). |
| **LFS Usage** | Public repos often have stricter limits on large file bandwidth. | Private repos allow for more controlled large file storage. |

### Switching Visibility
You can change a repo from Public to Private (and vice versa) in the **Settings** tab.
*   *Warning*: If you take a Public repo Private, you lose all your forks and stars.
*   *Warning*: If you take a Private repo Public, ensure you have scrubbed the history of all secrets and sensitive data!

## Interview Questions

**Q: When should an AI Engineer choose a Public repository?**
**A:** When they want to contribute to the open-source community, showcase their skills to potential employers, or share a tool/model that they want others to use and improve. It's the standard for "portfolio" projects.

**Q: What are the risks of a Public repository for an AI project?**
**A:** The main risks are accidental leakage of API keys (like OpenAI/Anthropic keys), exposing sensitive datasets that are not properly ignored, or losing intellectual property on a unique model architecture that has commercial value.

**Q: Can you have a private repository with 100 collaborators for free?**
**A:** Yes. GitHub currently allows unlimited collaborators on private repositories for free accounts. However, advanced features like "Branch Protection Rules" or "Code Owners" require a GitHub Pro or Team account for private repos.

**Q: Why is scrubbing history important when making a Private repo Public?**
**A:** If you ever committed a secret (like a database password or API key) in a private repo and then make it public, that secret is now visible in the **commit history**, even if it was deleted in the latest version. You must use tools like `git-filter-repo` or BFG Repo-Cleaner to permanently erase those secrets before going public.

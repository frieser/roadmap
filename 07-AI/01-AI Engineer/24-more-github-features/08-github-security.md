## Summary
GitHub Security provides a suite of tools to help developers find and fix vulnerabilities in their code. For AI Engineers, security is paramount when handling user data or deploying models that might be vulnerable to prompt injection or insecure dependency handling.

## Detailed Explanation

### Key Tools
1. **Dependabot**: Automatically detects out-of-date or vulnerable dependencies in your `requirements.txt` or `pyproject.toml` and creates PRs to update them.
2. **Secret Scanning**: Scans your code (and history) for accidentally committed secrets like API keys, tokens, or passwords.
3. **Code Scanning (CodeQL)**: Uses static analysis to find common coding errors and security vulnerabilities.
4. **Security Advisories**: A place to privately discuss and fix security vulnerabilities in your open-source projects.

### Security in AI Workflows
- **Dependency Poisoning**: AI libraries (like `transformers`) have many dependencies. Dependabot ensures you aren't using a version with a known exploit.
- **Dataset Security**: If you commit a script that downloads a dataset, ensure the URL isn't hardcoded with a sensitive SAS token or API key (Secret Scanning will catch this).

### Configuration
Most features are enabled via the **Security** tab of a repository. Code Scanning is typically set up as a GitHub Actions workflow.

## Interview Questions

**Q: What is CodeQL?**
**A:** CodeQL is the semantic analysis engine that powers GitHub code scanning. It treats code as data, allowing you to write queries to find complex patterns of vulnerabilities across your codebase.

**Q: What should you do if Secret Scanning alerts you that an API key was committed to your repository's history?**
**A:** 1. **Revoke** the key immediately. 2. **Rotate** it with a new one. 3. **Remove** the key from the Git history using a tool like `git-filter-repo` or BFG Repo-Cleaner (though revoking is the most critical step).

**Q: How does Dependabot know which versions of a library are vulnerable?**
**A:** It uses the **GitHub Advisory Database**, which is a collection of security advisories for many package managers, curated from public sources and community contributions.

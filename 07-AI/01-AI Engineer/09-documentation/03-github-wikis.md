## Summary
**GitHub Wikis** are a separate section of a repository for hosting long-form documentation that is too detailed for the README. They are useful for AI projects that require extensive setup guides, API references, or methodology deep-dives.

## Detailed Explanation
### **When to use a Wiki vs README**
- **README**: High-level overview, quick start, and essential info.
- **Wiki**: Detailed tutorials, internal design documents, comprehensive API documentation, and troubleshooting guides.

### **Features**
- **Versioning**: Wikis are themselves Git repositories, so you can track changes over time.
- **Sidebar and Footer**: For easy navigation between many pages.
- **Access**: Can be edited by anyone with write access to the repo (or everyone if configured).

### **AI Use Case**
An AI project might use a Wiki to host:
- A page for each supported dataset with detailed preprocessing steps.
- A "Hardware Guide" for setting up specific GPU clusters.
- A "Mathematical Appendix" explaining the derivations behind a custom loss function.

## Interview Questions
**Q: How is a GitHub Wiki different from a README.md file?**
**A:** A README is a single file in the root of the repository. A Wiki is a collection of pages that can be organized into a hierarchy, making it better for extensive, multi-page documentation.

**Q: Can you version control a GitHub Wiki?**
**A:** Yes, every Wiki is a Git repository. You can clone it locally, make changes, and push them back just like a normal repo.

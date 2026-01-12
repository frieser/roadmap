# GitHub Releases

## Summary
GitHub Releases are built on top of Git tags but provide a way to package software, provide release notes, and attach binary assets. For AI Engineers, this is the primary way to distribute models and their accompanying artifacts.

## Detailed Explanation

### Features of a Release
- **Based on a Tag:** Every GitHub Release must be associated with a Git tag.
- **Release Notes:** Markdown-formatted description of what changed in this version.
- **Assets:** You can attach binary files (up to 2GB each by default) to a release.

### AI Engineering Use Case
1.  **Distributing Model Weights:** While code is in Git, large model files (`.bin`, `.pth`, `.onnx`) are best distributed as "Assets" attached to a GitHub Release.
2.  **Packaging Evaluation Results:** Attaching a PDF or HTML report of model performance metrics to the release.
3.  **Source Code Snapshots:** Automatically provides `.zip` and `.tar.gz` downloads of the repo at that tag.
4.  **Changelogs:** Documenting changes in model architecture, training data updates, or performance improvements.

### Workflow
1.  Push a tag to GitHub.
2.  Go to the "Releases" section of the repo.
3.  Click "Draft a new release" and select the tag.
4.  Write release notes and upload model weights.
5.  Publish.

## Interview Questions
1.  **What is the difference between a Git tag and a GitHub Release?**
    A Git tag is a Git feature (a pointer to a commit). A GitHub Release is a GitHub-specific feature that adds release notes, binary assets, and a GUI on top of a tag.
2.  **Where should you store 500MB model weights: in the Git repo itself or as a Release Asset?**
    As a Release Asset. Storing large binary files in Git makes the repo slow and bloated for everyone.
3.  **How can you automate GitHub Release creation?**
    Using GitHub Actions (e.g., `softprops/action-gh-release`) to automatically create a release and upload assets whenever a tag is pushed.

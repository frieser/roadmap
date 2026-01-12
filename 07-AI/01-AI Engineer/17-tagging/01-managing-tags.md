# Managing Git Tags

## Summary
Tags are used to mark specific points in Git history as being important. They are most commonly used to mark release points (e.g., `v1.0`, `v2.1-alpha`). For AI Engineers, tags are crucial for versioning models and the code that produced them.

## Detailed Explanation

### Types of Tags
1.  **Lightweight Tags:** Just a pointer to a specific commit (like a branch that doesn't change).
    ```bash
    git tag v1.0-lw
    ```
2.  **Annotated Tags:** Stored as full objects in the Git database. They contain the tagger name, email, date, and a tagging message. **This is the recommended way to tag releases.**
    ```bash
    git tag -a v1.0.0 -m "Release version 1.0.0 with ResNet-50 backbone"
    ```

### Managing Tags
- **List tags:** `git tag`
- **Search for tags:** `git tag -l "v1.8*"`
- **View tag details:** `git show v1.0.0`
- **Delete a local tag:** `git tag -d v1.0.0`

### AI Engineering Context
1.  **Model Versioning:** Tagging the code used to train a specific model version. This ensures that even months later, you can check out the exact code to reproduce a result.
2.  **Milestones:** Tagging commits used for a specific research paper submission (e.g., `cvpr2024-submission`).
3.  **Deployment:** Using semantic versioning (`vX.Y.Z`) tags to trigger CI/CD pipelines that build Docker images for model serving.

## Interview Questions
1.  **What is the difference between an annotated tag and a lightweight tag?**
    Annotated tags contain metadata (author, date, message) and are checksummed; lightweight tags are just pointers to a commit.
2.  **How do you tag a commit that you made several days ago?**
    `git tag -a v0.9 <commit-hash>`
3.  **Why use tags instead of branches for releases?**
    Branches are intended to be mutable and move forward. Tags are intended to be permanent, immutable pointers to a specific snapshot in time.

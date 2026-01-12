## Summary
Git Large File Storage (LFS) is an open-source Git extension that replaces large files (datasets, models, audio, video) with text pointers inside Git. The actual file content is stored on a remote server, keeping your repository size small and cloning speeds fast.

## Detailed Explanation
### **Why LFS?**
Standard Git struggles with large files because it stores every version of every file in the `.git` directory. If you commit a 1GB model and update it five times, your repository grows to 5GB+. LFS stores only the "pointer" in Git, downloading the large file only when it's checked out.

### **Basic Workflow**
1. **Installation**: `git lfs install` (once per machine).
2. **Tracking**: `git lfs track "*.pth"` (creates/updates `.gitattributes`).
3. **Normal Git Flow**: `git add`, `git commit`, `git push`.

### **AI Engineering Use Case**
- **Model Versioning**: Storing `.bin`, `.pt`, `.weights` files.
- **Dataset Management**: Storing `.csv`, `.jsonl`, or `.zip` data files that are too large for standard Git.

### **Commands**
- `git lfs ls-files`: List files currently tracked by LFS.
- `git lfs pull`: Download the actual content for the pointers (usually done automatically by `git checkout`).
- `git lfs migrate`: Convert existing large files in your history to LFS.

## Interview Questions
- **Q: How does Git LFS store files differently than standard Git?**
- **A:** Standard Git stores the full content of every version of a file. Git LFS stores a small text pointer in the repository and stores the actual large content on a separate server.

- **Q: What command do you use to start tracking a specific file type with LFS?**
- **A:** `git lfs track "*.extension"`.

- **Q: What happens if you clone a repository that uses LFS but you don't have the LFS extension installed?**
- **A:** You will only see the small text pointers instead of the actual large files.

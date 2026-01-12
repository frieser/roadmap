# Git LFS (Large File Storage)

## Summary
Git is not designed for large binary files (images, audio, datasets). Git LFS is an extension that replaces these large files with text pointers inside Git, while storing the file contents on a remote server.

## Detailed Explanation

### Usage
1.  Install LFS: `git lfs install`.
2.  Track files: `git lfs track "*.psd"`.
3.  Add `.gitattributes`: `git add .gitattributes`.

### How it works
When you checkout, LFS downloads the specific large files you need for that commit. This keeps the `git clone` fast because you don't download the entire history of every large file.

### Go-specific Context
If your Go project embeds static assets (like a React frontend build or ML models), consider using LFS so your repository size doesn't explode.

## Interview Questions
**Q: Does GitHub support LFS?**
**A:** Yes, but there are storage and bandwidth limits on free accounts.

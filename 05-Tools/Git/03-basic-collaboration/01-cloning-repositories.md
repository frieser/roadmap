# Cloning Repositories

## Summary
Cloning is the process of downloading a copy of an existing remote repository to your local machine. It downloads every version of every file for the history of the project.

## Detailed Explanation
`git clone` is typically the first command you run when contributing to an existing project.

### Usage
```bash
git clone <repository-url> [directory-name]
```

### Protocols
*   **HTTPS**: `https://github.com/user/repo.git`. Easy to use, requires username/token for writing.
*   **SSH**: `git@github.com:user/repo.git`. Requires SSH keys, more secure, no password prompts.

### What it does
1.  Initialize a new `.git` directory.
2.  Add a remote called `origin` pointing to the URL.
3.  Fetch all data.
4.  Check out the default branch (usually `main`).

### Go-specific Context
In Go, `go get` essentially performs a clone (or fetch) into the module cache.
However, if you want to work on a library, you clone it manually:
```bash
# Workflow to patch a library
git clone https://github.com/gin-gonic/gin.git
cd gin
# ... make changes ...
```

## Interview Questions
**Q: What is the difference between `git clone` and `git pull`?**
**A:** `git clone` creates a new local copy of a remote repo. `git pull` updates an *existing* local copy with the latest changes from the remote.

**Q: Can you clone into a specific directory?**
**A:** Yes, `git clone <url> <my-dir>`.

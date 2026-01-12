# Git Init

## Summary
The `git init` command creates a new empty Git repository or reinitializes an existing one. It is typically the first command run when starting a new project. It creates a hidden `.git` directory which contains all the metadata and history for the repository.

## Detailed Explanation
When you run `git init` in a directory, Git sets up the internal plumbing required to track versions.

### Usage
```bash
# Initialize a repo in the current directory
git init

# Initialize a repo in a specific new directory
git init my-project
```

### What happens under the hood?
Git creates a `.git` folder containing:
*   `objects/`: Where all content (blobs, trees, commits) is stored.
*   `refs/`: Pointers to commit objects (branches, tags).
*   `HEAD`: Points to the current branch.
*   `config`: Local configuration for this repository.

### Go-specific Context
When starting a new Go project, you typically run two init commands:
1.  `go mod init module-name` (Initializes Go dependency management)
2.  `git init` (Initializes version control)

```bash
mkdir my-go-app
cd my-go-app
go mod init github.com/user/my-go-app
git init
git add go.mod
git commit -m "Initial commit"
```

## Interview Questions
**Q: What is stored in the `.git` directory?**
**A:** It stores the repository database (objects), references (branches/tags), configuration, hooks, and the staging area (index).

**Q: Can you run `git init` on a directory that is already a Git repo?**
**A:** Yes, it will reinitialize the repository. It won't overwrite things that are already there (like history), but it might pick up newly added templates.

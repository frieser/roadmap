# Why use Version Control?

## Summary
Version control software allows developers to track and manage changes to a project's code over time. It provides a history of every modification, enables multiple people to collaborate on the same codebase without overwriting each other's work, and offers a safety net to revert to previous states if bugs are introduced.

## Detailed Explanation
Version Control Systems (VCS) are essential tools for modern software development.

### Key Benefits
1.  **Collaboration**: Multiple developers can work on different parts of the same project simultaneously. The VCS manages merging their changes.
2.  **History & Auditability**: Every change is recorded with metadata: *who* made the change, *when*, and *why* (commit message). This allows you to trace the origin of a bug.
3.  **Risk Mitigation (Undo/Redo)**: If a new feature breaks the application, you can revert the codebase to the last working state instantly.
4.  **Branching & Experimentation**: Developers can create "branches" to test new ideas in isolation without affecting the main production code.
5.  **Backup**: In distributed systems like Git, every developer has a full copy of the project history, acting as a redundant backup.

### Types of VCS
*   **Local VCS**: Database on your computer (Revision Control System - RCS).
*   **Centralized VCS (CVCS)**: Single server contains all versioned files (Subversion, Perforce).
*   **Distributed VCS (DVCS)**: Clients fully mirror the repository (Git, Mercurial).

### Go-specific Context
In the Go ecosystem, version control is not just for code history; it is integral to dependency management.

*   **Go Modules**: When you run `go get github.com/gin-gonic/gin`, the Go toolchain uses Git to fetch the specific version of the library from its repository.
*   **Reproducible Builds**: `go.mod` and `go.sum` files lock dependencies to specific commit hashes or tags, ensuring that every build uses the exact same code, which is a core benefit of version control.

```go
// go.mod relies on VCS tags
module example.com/my-app

go 1.21

require (
    // This version v1.8.1 corresponds to a specific Git tag
    github.com/gorilla/mux v1.8.1
)
```

## Interview Questions
**Q: What is the main difference between a Centralized and a Distributed Version Control System?**
**A:** In a Centralized VCS (like SVN), the history is stored on a single central server; if the server goes down, no one can commit. In a Distributed VCS (like Git), every developer has a full copy of the repository (history and all) on their local machine, allowing for offline work and redundancy.

**Q: Why is version control important for a solo developer?**
**A:** It acts as an unlimited "undo" button, allows for safe experimentation via branches, and provides context on why certain code changes were made months or years ago.

**Q: What does it mean to "revert" a change?**
**A:** Reverting means undoing the effects of a specific commit or set of commits, returning the code to a previous state where the unwanted changes did not exist.

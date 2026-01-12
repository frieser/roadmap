# Git vs Other VCS

## Summary
Git is a **Distributed Version Control System (DVCS)**, distinguishing it from older Centralized systems like Subversion (SVN) or CVS. The key differentiators are that Git stores data as snapshots (not differences), every developer has a full repository copy, and it is optimized for rapid branching and merging.

## Detailed Explanation

### 1. Distributed vs. Centralized
*   **Centralized (SVN, CVS, Perforce):** A single server holds the file history. Developers check out a specific version. If the server is down, you cannot save changes or look at history.
*   **Distributed (Git, Mercurial):** Every client mirrors the repository. You can commit, view history, and branch entirely offline.

### 2. Snapshots vs. Deltas
*   **Other VCS:** often store data as a list of file-based changes (deltas). To build the current version, the system calculates changes from the beginning.
*   **Git:** thinks of data like a stream of **snapshots**. Every time you commit, Git takes a picture of what your files look like at that moment. If a file hasn't changed, it just links to the previous identical file. This makes Git extremely fast.

### 3. Data Integrity
Git uses SHA-1 hashing to identify everything. It is impossible to change the contents of any file or directory without Git knowing about it. This ensures data integrity and security.

### 4. Branching Model
Git's "killer feature" is its lightweight branching. Creating a branch is nearly instantaneous (41 bytes to create a new pointer), encouraging a workflow where you create a branch for every single feature or bug fix.

### Go-specific Context
Go was born in the era of Distributed VCS and assumes its existence.
*   **`go get`**: This command is VCS-aware. It can fetch code from Git, Mercurial (hg), SVN, or Bazaar, but Git is by far the most dominant.
*   **Vanity Imports**: Go allows custom import paths (like `rsc.io/quote`) which redirect the `go` tool to the underlying VCS repository (often Git).

```go
// When you import a package in Go:
import "github.com/google/uuid"

// The Go toolchain essentially performs a 'git clone' 
// (or fetch) of that repository to your local cache.
// This decentralized nature aligns perfectly with Git's philosophy.
```

## Interview Questions
**Q: How does Git store data differently than SVN?**
**A:** SVN stores changes as a list of file-based modifications (deltas). Git stores data as a series of snapshots of the entire project filesystem.

**Q: What is a "bare" repository in Git?**
**A:** A bare repository is a Git repository that has no working directory (no checked-out files). It is used purely for sharing and synchronization (like a central server), as opposed to a non-bare repo where you do your work.

**Q: Why is Git faster than centralized systems for branching?**
**A:** In Git, a branch is simply a lightweight movable pointer to a commit. Creating a branch just involves writing 40 characters (the SHA-1 hash) to a file, whereas in many centralized systems, branching involves copying the entire source tree.

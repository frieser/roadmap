---
tags: ['ai', 'roadmap']
---

## Summary
While Git is the industry standard today, it is part of a broader history of Version Control Systems. It is a **Distributed Version Control System (DVCS)**, which differs fundamentally from older **Centralized Version Control Systems (CVCS)** like SVN. Git's architecture ensures that every developer has a full copy of the project history, making it faster, more reliable, and better suited for the non-linear, experiment-heavy workflow of AI engineering.

## Detailed Explanation

### 1. CVCS vs. DVCS
The primary distinction in version control is where the history is stored.

#### Centralized VCS (e.g., SVN, Perforce)
*   There is a single central server that contains all the versioned files.
*   Developers "check out" files from the server, work on them, and "check in" (commit) back to the server.
*   **Major Weakness:** If the server goes down, no one can collaborate or save versions of their work. History is lost if the server's disk fails without backup.

#### Distributed VCS (e.g., Git, Mercurial)
*   Every developer "clones" the repository, creating a full mirror of the entire history on their local machine.
*   Commits are made locally first, then "pushed" to a remote server to share.
*   **Major Strength:** High resilience (every clone is a backup), fast local operations, and full offline capability.

### 2. Comparison Table

| Feature | Centralized (SVN) | Distributed (Git) |
| :--- | :--- | :--- |
| **Storage** | Central Server only | Every local machine has full history |
| **Offline Work** | Limited (only editing) | Full (commit, branch, merge, log) |
| **Speed** | Slow (requires network for most tasks) | Extremely Fast (mostly local) |
| **Branching** | Heavy/Slow (often copies files) | Lightweight/Instant (just a pointer) |
| **Reliability** | Single Point of Failure | No Single Point of Failure |

### 3. Snapshot vs. Delta
*   **SVN (Delta-based):** Stores the differences between files over time. To reconstruct a file, it adds up all the "patches."
*   **Git (Snapshot-based):** Thinks of data as a stream of snapshots. If a file hasn't changed, Git just stores a link to the previous identical file. This makes branching and merging much more efficient for complex AI codebases.

### 4. Why Git Won the AI/ML Era
Git's ability to handle massive numbers of branches and its deep integration with platforms like GitHub (where most open-source AI models live) made it the default choice. Its performance on large projects and robust merging logic are essential for teams managing complex neural network architectures and data pipelines.

## Interview Questions
**Q: What is the main difference between Git and SVN?**
**A:** Git is distributed (everyone has the full history), while SVN is centralized (only the server has the full history).

**Q: Why is "branching" better in Git than in Centralized VCS?**
**A:** In Git, a branch is just a tiny pointer to a commit, making it nearly instantaneous. In many CVCS, branching involves copying the actual project files into a new directory, which is slow and resource-intensive.

**Q: Does Git require an internet connection to commit code?**
**A:** No. Because it is distributed, you commit to your local repository. You only need a connection when you want to "Push" your changes to or "Pull" changes from a remote server.

**Q: What are some other VCS besides Git?**
**A:** Mercurial (Distributed), SVN (Subversion - Centralized), Perforce (Centralized - common in Game Dev), and BitKeeper (Historical).

**Q: What is the benefit of Git's "Snapshot" model over "Delta" storage?**
**A:** Snapshots make switching branches and merging much faster because the system doesn't have to calculate and apply a long string of file differences; it just looks at the state of the project at two different points.

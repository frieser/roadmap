# Forking vs Cloning

## Summary
While both involve copying a repository, **Forking** is a GitHub-specific concept that creates a server-side copy of a repo under your account. **Cloning** creates a local copy on your computer.

## Detailed Explanation

### Forking
*   **Where**: Happens on GitHub servers.
*   **Result**: You get `your-username/repo` which is a copy of `original-owner/repo`.
*   **Purpose**: To propose changes (Pull Requests) to a project you do not have write access to (Open Source).

### Cloning
*   **Where**: Happens on your local machine.
*   **Result**: A directory on your disk.
*   **Purpose**: To actually edit code.

### The OSS Workflow
1.  **Fork** `golang/go` to `myuser/go`.
2.  **Clone** `myuser/go` to local machine.
3.  **Edit** locally.
4.  **Push** to `myuser/go`.
5.  **Open PR** from `myuser/go` to `golang/go`.

## Interview Questions
**Q: Can I pull updates from the original repo into my fork?**
**A:** Yes, you configure the original repo as a remote (usually named `upstream`) and pull from it: `git pull upstream main`.

**Q: Do forks stay in sync automatically?**
**A:** No, you must manually sync them (though GitHub UI now has a "Sync Fork" button).

---
tags: ['ai', 'roadmap', 'git']
---

## Summary
`git fetch` is the command used to download the latest changes from a remote repository without integrating them into your local work. It is the "safe" way to see what your teammates have been doing. For AI Engineers, `fetch` is useful for reviewing updated training results or new model architectures on a remote branch before deciding to merge them into your own experimental workspace.

## Detailed Explanation

### `fetch` vs. `pull`
As an AI Engineer, you should understand the distinction:
*   **`git pull`**: `fetch` + `merge`. It downloads AND changes your code immediately.
*   **`git fetch`**: Only downloads. Your working directory remains exactly as it was.

### Why use `fetch`?
1.  **Safety**: You can see if a teammate's changes will break your current training run before you merge them.
2.  **Comparison**: You can compare your local code with the remote code.
3.  **Branch Discovery**: If someone creates a new branch (e.g., `experiment-gan-v2`), `fetch` will make that branch visible to you locally.

### Fetch Workflow
```bash
# 1. Download all updates from origin
git fetch origin

# 2. See what changes were downloaded
git log main..origin/main

# 3. Compare your local file with the remote version
git diff main origin/main -- train.py

# 4. (Optional) If you like the changes, merge them
git merge origin/main
```

### Understanding Remote-Tracking Branches
When you fetch, Git updates your **remote-tracking branches** (like `origin/main`). These are pointers that show where the branches were on the remote server the last time you communicated with it. You cannot move these pointers yourself; only `fetch` or `push` can update them.

## Interview Questions

**Q: Does `git fetch` change your local files?**
**A:** No. `git fetch` only updates the remote-tracking branches in your `.git` directory. It does not touch your working directory or your local branches.

**Q: How do you see the differences between your local `main` branch and the remote `main` branch after fetching?**
**A:** You can use `git diff main origin/main`. This shows the code differences between your current local state and the state you just downloaded from the remote.

**Q: Why is `git fetch` often preferred over `git pull` in professional environments?**
**A:** `git fetch` is non-destructive and allows for inspection. It gives the engineer a chance to review the incoming changes, check for potential conflicts, and run tests before altering their local workspace. `git pull` can lead to messy, unexpected merge conflicts if you aren't prepared.

**Q: How do you fetch changes from all registered remotes at once?**
**A:** Run `git fetch --all`. This is useful if you are tracking multiple remotes, such as a main company repository and a personal fork.

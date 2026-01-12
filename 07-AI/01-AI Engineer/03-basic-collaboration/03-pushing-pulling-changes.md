---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Pushing and Pulling are the primary methods for synchronizing your local Git repository with a remote server. **Pushing** uploads your local commits to the remote, while **Pulling** downloads changes from the remote and merges them into your current branch. For AI Engineers, these operations are vital for sharing model improvements, collaborating on dataset scripts, and ensuring that all team members are working with the latest codebase.

## Detailed Explanation

### 1. Pushing Changes (`git push`)
Once you've committed your changes locally, you need to "push" them to share them with others.
```bash
# Push the 'main' branch to 'origin'
git push origin main

# Set the upstream (tracking) the first time you push a new branch
git push -u origin feature/new-loss-function
```
*   **Pre-condition**: You must have a clean history (usually) and permission to write to the remote.
*   **AI Context**: Pushing training logs or updated `.yaml` configs to share experiment results.

### 2. Pulling Changes (`git pull`)
If a teammate has updated the preprocessing script, you need to "pull" those changes.
```bash
git pull origin main
```
`git pull` is actually a combination of two commands:
1.  **`git fetch`**: Downloads the data from the remote.
2.  **`git merge`**: Integrates the downloaded changes into your current local branch.

### 3. Handling Rejections
If the remote has commits that you don't have locally, Git will reject your push.
*   **The Solution**: Pull the changes first, resolve any conflicts, and then push again.
*   **The Shortcut (Avoid if possible)**: `git push --force` overwrites the remote history. **Never do this on shared branches (like main)** as it can delete your teammates' work.

### 4. Pushing Large Files (Git LFS)
When pushing AI models, standard `git push` handles the LFS pointer files, while the LFS extension handles the actual upload of the large binary files to the LFS server.
```bash
# Standard command works, but LFS runs in the background
git push origin main
```

## Interview Questions

**Q: What is the difference between `git pull` and `git fetch`?**
**A:** `git fetch` only downloads the latest changes from the remote but does not modify your working directory. `git pull` performs a `git fetch` AND immediately tries to merge those changes into your current branch. `fetch` is safer if you want to review changes before merging.

**Q: What does the `-u` flag do in `git push -u origin main`?**
**A:** The `-u` (or `--set-upstream`) flag links your local branch to the remote branch. Once linked, you can simply run `git push` or `git pull` without specifying the remote or branch name in the future.

**Q: Why might a `git push` be rejected even if you have write access?**
**A:** It is usually rejected because the remote repository contains commits that you do not have locally (i.e., your local branch is "behind" the remote). You must `git pull` (or `fetch` and `merge/rebase`) to sync your history before Git allows you to push.

**Q: When is it acceptable to use `git push --force`?**
**A:** It is generally only acceptable on a private feature branch where you are the only person working. For example, if you rebased your feature branch to clean up the history before a Pull Request, you would need to force push to update the remote.

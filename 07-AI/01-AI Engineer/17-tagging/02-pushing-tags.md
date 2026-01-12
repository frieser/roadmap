# Pushing Tags to a Remote

## Summary
By default, the `git push` command does not transfer tags to remote servers. You must explicitly push tags after creating them locally.

## Detailed Explanation

### Pushing Specific Tags
```bash
git push origin v1.0.0
```

### Pushing All Local Tags
```bash
git push origin --tags
```

### Deleting Remote Tags
To remove a tag from the server (use with caution):
```bash
git push origin --delete v1.0.0
```

### AI Engineering Context
1.  **Collaboration:** Pushing tags allows team members to pull the exact same code version you used for a successful experiment.
2.  **Automated Build Triggers:** Many CI tools (GitHub Actions, GitLab CI) are configured to only run "production build" jobs when a new tag is pushed.
3.  **Release Transparency:** Pushing tags to GitHub makes them appear in the "Tags" tab, making it easy for stakeholders to download specific source code versions.

## Interview Questions
1.  **Does `git push` include tags by default?**
    No.
2.  **What is the command to push all your local tags at once?**
    `git push origin --tags`
3.  **Why might you want to avoid `git push --tags` in a large team?**
    It might push local "junk" or temporary tags that you haven't cleaned up, cluttering the shared repository. It's often better to push specific, intended tags.

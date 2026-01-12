---
tags: ['ai', 'roadmap', 'git']
---

## Summary
Viewing commit history is the process of exploring the chronological record of changes in a repository. For AI Engineers, this is vital for tracking experiment iterations, understanding when a specific hyperparameter was changed, or identifying which commit introduced a bug in the data pipeline. Git provides powerful tools like `git log` and `git show` to navigate this history efficiently.

## Detailed Explanation

### The `git log` Command
The primary tool for viewing history.
```bash
# Basic log (shows author, date, message)
git log

# Concise one-line view (great for a quick overview)
git log --oneline

# Visual graph view (shows branches and merges)
git log --oneline --graph --all
```

### Filtering History
In large AI projects, finding a specific change can be difficult. Use filters:
```bash
# See changes to a specific file (e.g., the model definition)
git log -- src/model.py

# Search for a keyword in commit messages (e.g., "learning rate")
git log --grep="learning rate"

# See changes by a specific author
git log --author="Alex"

# See changes since a specific time
git log --since="2 weeks ago"
```

### Inspecting Specific Changes
To see what actually changed inside a commit:
```bash
# Show the details and diff of a specific commit
git show <commit_hash>

# Show the diff of the most recent commit
git show HEAD
```

### Comparing Commits
To see how the project evolved between two training runs:
```bash
# Compare two commits
git diff <commit_hash_1> <commit_hash_2>

# Compare your current work with a specific previous version
git diff <commit_hash> -- train.py
```

### Using Git Blame
If you find a line of code that looks wrong, use `blame` to see who changed it last and why:
```bash
git blame src/train.py
```

## Interview Questions

**Q: What is the benefit of using `git log --oneline --graph --all`?**
**A:** This command provides a high-level, visual representation of the entire repository history. The `--graph` shows how branches diverge and merge, `--all` shows all branches (not just the current one), and `--oneline` keeps it compact. This is essential for understanding complex project structures.

**Q: How can you find the commit that deleted a specific file?**
**A:** You can use `git log -- <file_path>` to see all commits that affected that path, or use `git log --diff-filter=D --summary` to specifically list commits where files were deleted.

**Q: What does `git show` do?**
**A:** `git show` displays the metadata and the content differences (the diff) for a specific Git object, most commonly a commit. It is the quickest way to see exactly what code was added or removed in a single snapshot.

**Q: Why is `git blame` useful for an AI Engineer?**
**A:** `git blame` shows line-by-line who last modified a file and in which commit. In AI engineering, this helps identify who changed a critical model parameter or a data preprocessing step, allowing you to find the associated commit message and understand the reasoning behind the change.

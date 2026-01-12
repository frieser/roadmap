# .gitignore

## Summary
The `.gitignore` file tells Git which files or directories to intentionally ignore. These files will not be tracked, will not show up in `git status`, and will not be committed.

## Detailed Explanation

### Syntax
*   `#`: Comments.
*   `*`: Wildcard (matches any characters).
*   `/`: Directory separator (leading slash matches root).
*   `!`: Negate (do not ignore this file).

### Go-specific .gitignore
A standard Go project should ignore build artifacts and dependency directories (if vendoring is not used or handled differently).

```gitignore
# Binaries
# Ignore all files ending in .exe or .test
*.exe
*.test
*.out

# Output directory (if you build to dist/)
/dist/
/bin/

# Dependency directories (optional, depending on strategy)
vendor/

# Go workspace file
go.work

# OS specific
.DS_Store
Thumbs.db

# Editor specific
.vscode/
.idea/
```

### Global Ignore
You can set a global gitignore for your user (e.g., to always ignore `.DS_Store` across all projects):
```bash
git config --global core.excludesfile ~/.gitignore_global
```

## Interview Questions
**Q: If a file is already tracked by Git, will adding it to `.gitignore` stop tracking it?**
**A:** No. You must explicitly remove it from the index using `git rm --cached <file>`. `.gitignore` only applies to untracked files.

**Q: How do you ignore all `.log` files except `important.log`?**
**A:**
```gitignore
*.log
!important.log
```

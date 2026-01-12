# Git Log Options

## Summary
`git log` is highly customizable. You can filter the history to find exactly what you are looking for.

## Detailed Explanation

### Filtering
*   **Count**: `-n 5` (Last 5 commits).
*   **Date**: `--since="2 weeks ago"` or `--until="2023-01-01"`.
*   **Author**: `--author="John"`.
*   **File**: `git log -- path/to/file.go`.
*   **Search Content**: `-S "functionName"` (Pickaxe search - finds commits that added/removed this string).

### Formatting
*   `--oneline`: Hash + Subject.
*   `--stat`: Show modified files and line counts.
*   `--graph`: ASCII tree.
*   `--pretty=format:"%h - %an, %ar : %s"`: Custom string format.

### Go-specific Context
To find when a specific Go dependency was added to `go.mod`:
```bash
git log -S "github.com/gin-gonic/gin" go.mod
```

## Interview Questions
**Q: How do you see the changes introduced by each commit in the log?**
**A:** `git log -p` (patch).

**Q: How do you see commits that changed a specific function?**
**A:** `git log -L :funcName:file.go` (Line log search).

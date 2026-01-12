---
---

## Summary
The `find` command is the Swiss Army knife for searching for files and directories in a filesystem hierarchy based on various criteria (name, size, modification time, permissions) and performing actions on them.

## Detailed Explanation

### Syntax
`find <path> <criteria> <action>`

### Common Criteria
*   **Name**: `-name "*.go"` (Case sensitive), `-iname` (Case insensitive).
*   **Type**: `-type f` (File), `-type d` (Directory).
*   **Time**:
    *   `-mtime -7`: Modified in last 7 days.
    *   `-mmin -10`: Modified in last 10 minutes.
*   **Size**: `-size +100M` (Larger than 100MB).
*   **Permissions**: `-perm 644`.

### Actions
*   `-print` (Default): Print path.
*   `-delete`: Delete matches.
*   `-exec`: Run a command on matches.
    *   `find . -name "*.tmp" -exec rm {} \;` (`{}` is the placeholder for the file).

### `-exec` vs `xargs`
*   `-exec rm {} \;`: Spawns a new `rm` process for **every** file. (Slow).
*   `| xargs rm`: Batches arguments and runs `rm` once (or few times). (Fast).

## Go-Specific Context/Examples

Go's `path/filepath` package provides `Walk` and `WalkDir` to replicate `find` functionality programmatically.

### Example: Walking a Directory Tree

```go
package main

import (
	"fmt"
	"io/fs"
	"path/filepath"
)

func main() {
	root := "."
	filepath.WalkDir(root, func(path string, d fs.DirEntry, err error) error {
		if err != nil {
			return err
		}
		// Equivalent to: find . -name "*.go"
		if !d.IsDir() && filepath.Ext(path) == ".go" {
			fmt.Println("Found Go file:", path)
		}
		return nil
	})
}
```

## Interview Questions

**Q: How do you find files modified in the last 24 hours?**
**A:** `find . -mtime -1` (Technically checks 24h blocks, so -1 means < 24h ago).

**Q: Why use `find . -print0 | xargs -0`?**
**A:** To handle filenames with **spaces**. By default, `xargs` splits on whitespace, so "My File.txt" becomes two arguments "My" and "File.txt". `-print0` separates filenames with a null byte (`\0`), and `-0` tells xargs to use null as the delimiter, ensuring safety.

**Q: How to ignore the `.git` directory while searching?**
**A:** Use `-prune`.
`find . -path ./git -prune -o -name "*.txt" -print`
(If path is ./git, prune it (don't descend); otherwise, check name and print).

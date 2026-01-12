---
---

## Summary
File test operators are unary operators used to check the existence, type, and permissions of files and directories. They are fundamental for writing robust scripts that interact with the filesystem.

## Detailed Explanation

### Existence & Type
*   `-e file`: Exists (regardless of type).
*   `-f file`: Exists and is a regular **file**.
*   `-d file`: Exists and is a **directory**.
*   `-L file`: Exists and is a **symbolic link**.
*   `-s file`: Exists and size is > 0 (Non-empty).

### Permissions
*   `-r file`: Readable by current user.
*   `-w file`: Writable by current user.
*   `-x file`: Executable by current user.

### Comparison
*   `file1 -nt file2`: Newer Than (modification time).
*   `file1 -ot file2`: Older Than.

## Go-Specific Context/Examples

In Go, you use `os.Stat` to get file info and check errors.

### Analogy: Check if file exists
**Bash**:
```bash
if [ -e "config.json" ]; then ...
```
**Go**:
```go
if _, err := os.Stat("config.json"); err == nil {
    // Exists
} else if os.IsNotExist(err) {
    // Does not exist
}
```

### Analogy: Check if dir
**Bash**: `[ -d "logs" ]`
**Go**: `fi, _ := os.Stat("logs"); fi.IsDir()`

## Interview Questions

**Q: How do you check if a directory does NOT exist and create it?**
**A:** `if [ ! -d "$DIR" ]; then mkdir -p "$DIR"; fi`.

**Q: Does `-f` return true for a symlink?**
**A:** Only if the symlink points to a valid regular file. If you want to check if it's specifically a link (regardless of target), use `-L`.

**Q: What does `-s` check?**
**A:** It checks if the file exists AND has a size greater than zero. Useful to verify that a download or log file actually contains data.

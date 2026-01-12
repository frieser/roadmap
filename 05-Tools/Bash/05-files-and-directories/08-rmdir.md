---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# rmdir (Remove Directory)

## Summary

The `rmdir` command removes empty directories. Unlike `rm -r`, it only works on directories that contain no files or subdirectories. This makes it safer for removing directories that should be empty, as it will fail if they unexpectedly contain files.

## Detailed Explanation

### Basic Usage

```bash
# Remove empty directory
rmdir mydir

# Remove multiple directories
rmdir dir1 dir2 dir3

# Fails if directory is not empty
rmdir /tmp/full_directory
# rmdir: failed to remove '/tmp/full_directory': Directory not empty
```

### Options

```bash
# -p: Remove parent directories if empty
mkdir -p a/b/c
rmdir -p a/b/c     # Removes c, then b, then a (if all empty)

# -v: Verbose
rmdir -v emptydir
# rmdir: removing directory, 'emptydir'

# --ignore-fail-on-non-empty
rmdir --ignore-fail-on-non-empty dir
# No error even if not empty (just doesn't remove)
```

### rmdir vs rm -r

| Command | Empty Dir | Non-Empty Dir | Safety |
|---------|-----------|---------------|--------|
| `rmdir` | Removes | Fails | Safe |
| `rm -r` | Removes | Removes all | Dangerous |
| `rm -d` | Removes | Fails | Safe |

```bash
# Use rmdir when you expect directory to be empty
rmdir ./old_cache

# Use rm -r when you want to delete everything
rm -r ./old_project

# rm -d is like rmdir
rm -d emptydir
```

### Common Patterns

```bash
# Remove directory structure (all must be empty)
rmdir -p project/src/main

# Clean up empty directories
find . -type d -empty -delete

# Check if directory is empty first
if [ -z "$(ls -A "$dir")" ]; then
    rmdir "$dir"
else
    echo "Directory not empty"
fi

# Remove only if empty (ignore otherwise)
rmdir --ignore-fail-on-non-empty "$dir" 2>/dev/null
```

## Interview Questions

**Q: What is the difference between `rmdir` and `rm -r`?**
**A:** `rmdir` only removes empty directories and fails safely on non-empty ones. `rm -r` removes directories and all their contents recursively. Use `rmdir` when directories should be empty for safety.

**Q: How do you remove parent directories with rmdir?**
**A:** Use `rmdir -p a/b/c`. This removes `c`, then `b`, then `a`, but only if each becomes empty after removing its child.

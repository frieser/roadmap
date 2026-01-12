---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# mkdir (Make Directory)

## Summary

The `mkdir` command creates new directories. With the `-p` flag, it can create parent directories as needed and won't fail if the directory already exists. It's fundamental for file organization and scripting automation.

## Detailed Explanation

### Basic Usage

```bash
# Create single directory
mkdir myproject

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories (fails without -p)
mkdir project/src/main    # Error if project/ doesn't exist
```

### Common Options

```bash
# -p: Create parent directories as needed
mkdir -p project/src/main/java
mkdir -p ~/projects/new-app/{src,bin,lib,docs}

# -p also ignores existing directories
mkdir -p existing_dir    # No error

# -v: Verbose - show what's being created
mkdir -pv project/src
# mkdir: created directory 'project'
# mkdir: created directory 'project/src'

# -m: Set permissions
mkdir -m 755 public
mkdir -m 700 private
```

### Brace Expansion for Structure

```bash
# Create project structure in one command
mkdir -p project/{src/{main,test},docs,bin,lib}

# Creates:
# project/
# ├── bin/
# ├── docs/
# ├── lib/
# └── src/
#     ├── main/
#     └── test/
```

### Common Patterns in Scripts

```bash
#!/bin/bash

# Safe directory creation
mkdir -p "$TARGET_DIR" || exit 1

# Create with error handling
if ! mkdir -p "$DIR" 2>/dev/null; then
    echo "Failed to create $DIR" >&2
    exit 1
fi

# Create temp directory
TMPDIR=$(mktemp -d)
trap "rm -rf $TMPDIR" EXIT
```

## Interview Questions

**Q: What does `mkdir -p` do?**
**A:** Creates parent directories as needed and doesn't fail if the directory exists. `mkdir -p a/b/c` creates `a`, `a/b`, and `a/b/c` if they don't exist.

**Q: How do you create multiple directories at once?**
**A:** Use brace expansion: `mkdir -p project/{src,bin,lib}` or list them: `mkdir dir1 dir2 dir3`.

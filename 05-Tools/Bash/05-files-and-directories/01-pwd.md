---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# pwd (Print Working Directory)

## Summary

The `pwd` command prints the absolute path of the current working directory. It's one of the most basic yet essential commands for navigation and orientation in the filesystem. Understanding the difference between physical and logical paths is important when working with symbolic links.

## Detailed Explanation

### Basic Usage

```bash
# Print current directory
pwd
# /home/user/projects

# After navigation
cd /var/log
pwd
# /var/log
```

### Options

```bash
# -L: Logical path (default) - follows symlinks
pwd -L

# -P: Physical path - shows real location
pwd -P

# Example with symlink
ln -s /var/log /home/user/logs
cd /home/user/logs
pwd -L    # /home/user/logs (logical)
pwd -P    # /var/log (physical)
```

### pwd vs $PWD

```bash
# Built-in variable (faster, same result)
echo $PWD
# /home/user/projects

# Both are equivalent for most purposes
[ "$(pwd)" = "$PWD" ] && echo "Same"

# $OLDPWD stores previous directory
cd /tmp
echo $OLDPWD   # /home/user/projects
cd -           # Returns to OLDPWD
```

### Use in Scripts

```bash
#!/bin/bash

# Save current directory
original_dir=$(pwd)

# Do work in different directory
cd /tmp
# ... operations ...

# Return to original
cd "$original_dir"

# Or use pushd/popd
pushd /tmp > /dev/null
# ... operations ...
popd > /dev/null
```

## Interview Questions

**Q: What is the difference between `pwd -L` and `pwd -P`?**
**A:** `-L` (logical) shows the path including symlinks as you traversed them. `-P` (physical) resolves symlinks to show the actual filesystem location. Default is `-L`.

**Q: How do you get the directory of the current script?**
**A:** Use `script_dir=$(cd "$(dirname "$0")" && pwd)`. This handles relative paths and symlinks. Just `$(dirname "$0")` might give a relative path.

---
---

## Summary
Bash supports recursive functions (functions calling themselves). This is useful for traversing directory trees or mathematical calculations, though Bash is slower and has a lower stack limit than compiled languages.

## Detailed Explanation

### Example: Factorial
```bash
factorial() {
    if [ "$1" -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $(( $1 - 1 )))
        echo $(( $1 * prev ))
    fi
}
```

### Example: Directory Traversal
```bash
walk() {
    for file in "$1"/*; do
        if [ -d "$file" ]; then
            walk "$file" # Recurse
        else
            echo "File: $file"
        fi
    done
}
```

### Stack Limits
Bash does not have TCO (Tail Call Optimization). Deep recursion will eventually hit a stack limit or memory limit, crashing the shell.

## Go-Specific Context/Examples

Recursion works the same way in Go but is much faster and safer.

### Analogy
**Bash**: Uses `local` variables and subshells `$()` to manage state, which is expensive.
**Go**: Uses stack frames efficiently. `filepath.Walk` is the standard iterative/recursive hybrid for file systems.

## Interview Questions

**Q: Why is recursion in Bash often slow?**
**A:** Because functions often rely on subshells (like `$(recursive_call)`) to capture output. Spawning a subshell for every recursive step adds massive overhead compared to a simple function call in C or Go.

**Q: How do you prevent infinite recursion?**
**A:** Ensure there is a **Base Case** (e.g., `if [ $1 -le 1 ]`) that stops the recursion. Without it, the script will run until the system kills it or it runs out of memory.

**Q: Is it better to use `find` or a recursive Bash function?**
**A:** Always use `find` (or `fd`). It is written in C, highly optimized, and handles edge cases (symlink loops, permissions) much better than a hand-written Bash loop.

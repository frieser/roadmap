---
---

## Summary
ShellCheck is a static analysis tool (Linter) for shell scripts. It is the gold standard for Bash code quality. It detects syntax errors, semantic problems, and common pitfalls that `bash -n` misses, such as quoting issues, unused variables, or misleading comparisons.

## Detailed Explanation

### Features
*   **Quoting**: Warns if you forget to quote variables (`$VAR` vs `"$VAR"`).
*   **Logic**: Warns about `[ $foo = bar ]` (should be `==` or quoted).
*   **Portability**: Warns if you use Bashisms (`[[ ]]`) in a script starting with `#!/bin/sh`.
*   **Common Bugs**: Forgetting that `cd` can fail (`cd dir; rm *`).

### Codes
Errors have codes like `SC2006` (Use `$(...)` instead of legacy backticks `` `...` ``). You can look these up on the ShellCheck wiki.

## Go-Specific Context/Examples

ShellCheck is basically `golangci-lint` or `staticcheck` for Bash.

### Analogy
*   **Bash**: `shellcheck script.sh`
*   **Go**: `staticcheck ./...`

Both tools analyze the AST (Abstract Syntax Tree) to find patterns that are syntactically valid but likely bugs.

## Interview Questions

**Q: Why does ShellCheck recommend double-quoting every variable?**
**A:** To prevent **Word Splitting** and **Globbing**. If `$FILENAME` is `My File.txt`, `rm $FILENAME` tries to delete `My` and `File.txt`. `rm "$FILENAME"` deletes the intended file.

**Q: How do you suppress a specific ShellCheck warning?**
**A:** Add a comment above the line: `# shellcheck disable=SC2006`.

**Q: What is the most dangerous common bug ShellCheck catches?**
**A:** Failing to check if `cd` succeeded before running destructive commands.
*   Bad: `cd /tmp/work; rm -rf *` (If `cd` fails, you wipe your current dir!)
*   Fix: `cd /tmp/work || exit 1; rm -rf *`

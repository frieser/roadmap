---
---

## Summary
`bash -n` (No Exec) is a syntax checking mode. It reads the script and checks for syntax errors (like unclosed quotes, missing `fi` or `done`) but does **not** execute any commands. It is a "Dry Run" for syntax validity.

## Detailed Explanation

### Usage
`bash -n script.sh`

### What it Detects
*   Missing quotes: `echo "hello`
*   Missing control structures: `if [ condition ]; then echo hi` (Missing `fi`)
*   Syntax errors in loop definitions.

### What it Misses
*   Logic errors.
*   Typographical errors in command names (e.g., `eccho` instead of `echo`).
*   Runtime errors (e.g., file not found).

## Go-Specific Context/Examples

This is equivalent to `go vet` or `go build` (without running the binary).

### Analogy
*   **Bash**: `bash -n script.sh`
*   **Go**: `go vet ./...` or `go build -o /dev/null`

## Interview Questions

**Q: Why doesn't `bash -n` catch misspelled variables?**
**A:** In Bash, undefined variables are valid (they just expand to empty strings). This is a feature, not a syntax error. To catch this, you need execution time checks like `set -u` (Nounset).

**Q: Can you put `set -n` inside the script?**
**A:** Yes, but once you set it, the script stops executing commands immediately. You can't turn it off (`set +n`) via a command because... commands are no longer being executed! It's mostly useful as a CLI argument.

**Q: Does `bash -n` verify that external commands exist?**
**A:** No. `bash -n` only checks the *shell grammar*. It assumes `my_custom_command` is a valid command that will exist at runtime.

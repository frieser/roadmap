---
---

## Summary
Sourcing a script (using `source` or `.`) executes the commands **in the current shell process**, rather than spawning a new subshell. This allows the script to modify the current environment (e.g., setting environment variables, changing directory, defining functions) that persist after the script finishes.

## Detailed Explanation

### Syntax
*   `source script.sh` (Bash specific, more readable).
*   `. script.sh` (POSIX standard, portable).

### Behavior
*   **New Process (`./script.sh`)**: Variables set inside die when script exits. `cd /tmp` only affects the script, not your terminal.
*   **Sourced (`source script.sh`)**: Variables set inside become part of your current shell. `cd /tmp` changes your terminal's directory.

### Use Cases
*   Loading configuration (`source .env`).
*   Activating virtual environments (`source venv/bin/activate`).
*   Loading library functions into your shell.

## Go-Specific Context/Examples

Go programs run in their own process. They cannot "source" a script to modify their own environment directly (e.g., `os.Setenv` only affects the Go process and its children, not the parent shell).

To "load" env vars from a file in Go, you must parse the file manually (like `godotenv`).

### Analogy
*   **Running**: `exec.Command` (Child process).
*   **Sourcing**: No direct equivalent in Go runtime, except importing a package.

## Interview Questions

**Q: Why does `cd` inside a script not change my terminal's directory?**
**A:** Because scripts run in a **subshell**. The subshell changes its directory and then exits, returning you to the parent shell (your terminal), which remained unchanged. To change the parent, you must `source` the script.

**Q: What happens if you `exit` inside a sourced script?**
**A:** It exits the **current shell** (your terminal window closes or SSH session disconnects). Always use `return` instead of `exit` inside scripts intended to be sourced.

**Q: Is `source` available in `sh`?**
**A:** No, `source` is a Bash keyword. In strict POSIX `sh`, you must use the dot `.` operator.

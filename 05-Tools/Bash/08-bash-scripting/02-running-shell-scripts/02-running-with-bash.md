---
---

## Summary
You can execute a shell script by explicitly passing it as an argument to the interpreter, e.g., `bash script.sh`. This method overrides the file's execute permissions (doesn't need `+x`) and ignores the Shebang line. It is useful for debugging or running scripts in restricted environments.

## Detailed Explanation

### Usage
`bash [options] script.sh`

### Common Options
*   **`-x` (Debug)**: Print each command before executing it (Trace mode). Great for debugging logic.
*   **`-n` (No Exec)**: Check syntax errors but do not run the script.
*   **`-e` (Exit on Error)**: Exit immediately if a command exits with a non-zero status.

### Implications
*   **Shebang Ignored**: Running `bash script.py` will try to execute Python code as Bash commands (and fail).
*   **Permissions Ignored**: You only need **Read** permission (`r`) on the file, not Execute (`x`).

## Go-Specific Context/Examples

When orchestrating scripts from Go, executing via `bash -c` allows you to run inline scripts or handle complex piping without an external file.

### Example: Running Inline Bash from Go
```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	// Run a complex bash pipeline directly
	cmd := exec.Command("bash", "-c", "ls -la | grep .go")
	
	output, _ := cmd.CombinedOutput()
	fmt.Println(string(output))
}
```

## Interview Questions

**Q: What is the difference between `./script.sh` and `bash script.sh`?**
**A:** `./script.sh` requires `+x` permission and uses the interpreter defined in the shebang. `bash script.sh` does not require `+x` and forces the use of `bash`, ignoring the shebang.

**Q: How do you debug a script that is crashing silently?**
**A:** Run it with `bash -x script.sh`. This prints every command to stderr with its expanded arguments before execution, allowing you to trace exactly where it fails.

**Q: What does `set -e` inside a script do?**
**A:** It has the same effect as running with `bash -e`. It forces the script to abort immediately if any command fails (returns non-zero), preventing cascading errors in deployment scripts.

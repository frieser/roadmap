---
---

# Bash Scripting for DevOps

Bash (Bourne Again SHell) is the de facto standard CLI scripting language for Linux systems. For DevOps engineers, it is the "glue" code used for bootstrapping servers, wrapping deployment commands, and performing ad-hoc system maintenance tasks.

## Summary

Bash scripting allows for the automation of command-line tasks. Key concepts include **variables**, **conditionals** (`if`, `case`), **loops** (`for`, `while`), and **functions**. In a DevOps context, writing robust Bash scripts requires strict error handling (using `set -e`), proper logging, and understanding input/output redirection. While powerful, complex logic should often be promoted to a higher-level language like **Go** or Python for maintainability.

## Detailed Explanation

### 1. Essential Best Practices
*   **Shebang**: Always start with `#!/bin/bash` to ensure the correct interpreter.
*   **Safety Flags** (`set -euo pipefail`):
    *   `-e`: Exit immediately if a command exits with a non-zero status.
    *   `-u`: Treat unset variables as an error.
    *   `-o pipefail`: Return the exit status of the last command in the pipe that failed, not just the last command.
*   **Quoting**: Always quote variables (`"$VAR"`) to prevent word splitting issues.

### 2. Core Constructs
*   **Variables**: `NAME="DevOps"` (No spaces around `=`).
*   **Conditionals**:
    ```bash
    if [ -f "/etc/config" ]; then
        echo "Config exists"
    fi
    ```
*   **Loops**:
    ```bash
    for ip in $(cat servers.txt); do
        ssh "$ip" "uptime"
    done
    ```

### 3. Transition to Go
Bash becomes unmaintainable as scripts grow. Go is an excellent replacement because it produces static binaries that are easy to distribute to servers without worrying about interpreter versions or dependencies.

---

## Go Implementation Example

There are two main ways to replace Bash with Go: using the standard `os/exec` package for raw power, or using a library like `bitfield/script` for a more "Bash-like" API.

### Approach 1: Standard Library (`os/exec`)
This approach gives you full control over execution, environment, and I/O.

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
)

func main() {
	// Equivalent to: ls -la | grep "json"
	
	// Command 1: ls -la
	cmdLS := exec.Command("ls", "-la")
	
	// Command 2: grep "json"
	cmdGrep := exec.Command("grep", "json")

	// Create a pipe between the two commands
	// The output of LS becomes the input of Grep
	reader, writer, err := os.Pipe()
	if err != nil {
		log.Fatal(err)
	}
	
	cmdLS.Stdout = writer
	cmdGrep.Stdin = reader
	
	// Grep output goes to the main process Stdout
	cmdGrep.Stdout = os.Stdout

	// Start both commands
	cmdLS.Start()
	cmdGrep.Start()
	
	// Close the writer side of the pipe so Grep knows input is finished
	writer.Close()
	
	// Wait for completion
	cmdLS.Wait()
	cmdGrep.Wait()
}
```

### Approach 2: Fluent API (`bitfield/script`)
For simple scripting tasks, the `script` library mimics the pipe syntax of Bash.

```go
package main

import (
	"fmt"
	"github.com/bitfield/script"
)

func main() {
	// Equivalent to: cat access.log | grep "500" | wc -l
	count, err := script.File("access.log").
		Match("500").
		CountLines()
		
	if err != nil {
		fmt.Println("Error:", err)
	}
	
	fmt.Printf("Found %d 500 errors\n", count)
}
```

## Interview Questions

**Q: What does `set -euo pipefail` do, and why is it important?**
**A:** It is a "strict mode" for Bash. `-e` stops the script on error, `-u` errors on undefined variables, and `-o pipefail` ensures the script fails if *any* command in a pipe fails (e.g., `cmd1 | cmd2`). Without this, a script might silently fail halfway through but continue executing destructive commands.

**Q: How do you check if a variable is set in Bash?**
**A:** You can use the syntax `if [ -z "$VAR" ]; then ... fi` to check if it's empty, or `if [ -n "$VAR" ]; then ... fi` to check if it's not empty. The `-u` flag helps catch cases where you reference a variable that was never defined.

**Q: Difference between `$@` and `$*`?**
**A:** Both refer to all script arguments. However, `"$@"` expands each argument as a separate quoted string (`"arg1" "arg2"`), preserving spaces within arguments. `"$*"` expands to a single string with all arguments joined by the first character of IFS (`"arg1 arg2"`). `"$@"` is almost always what you want.

**Q: When should you switch from Bash to Go (or Python)?**
**A:** You should switch when the script requires complex data structures (maps/arrays), parsing structured data (JSON/YAML), heavy unit testing, cross-platform compatibility (Windows/Linux), or when the script exceeds ~100 lines of logic.

---
---

## Summary
The exit status (or return code) of the last executed command is stored in the special variable `$?`. It is a numeric value between 0 and 255. By convention, **0 means Success**, and any non-zero value means Failure.

## Detailed Explanation

### Checking Status
You should check `$?` immediately after the command, because every subsequent command updates it.

```bash
ls /non/existent/dir
if [ $? -ne 0 ]; then
    echo "Command failed!"
fi
```

### Chaining
Operators like `&&` and `||` rely implicitly on the exit status.
*   `mkdir dir && cd dir` (Run `cd` only if `mkdir` returned 0).

## Go-Specific Context/Examples

In Go, when you run an external command, the exit code is hidden inside the error object.

### Example: Checking Exit Code in Go
```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	cmd := exec.Command("ls", "/missing")
	err := cmd.Run()

	if err != nil {
		// Attempt to get the exit code
		if exitError, ok := err.(*exec.ExitError); ok {
			fmt.Println("Exit Code:", exitError.ExitCode())
		}
	} else {
		fmt.Println("Success (Exit Code 0)")
	}
}
```

## Interview Questions

**Q: What happens if you exit with a code > 255?**
**A:** It wraps around modulo 256. `exit 256` results in code 0 (Success!). `exit 257` results in code 1. This can lead to dangerous bugs if you try to return huge numbers as exit codes.

**Q: How do you capture the exit status of a command in a pipe?**
**A:** `$?` only gives the status of the *last* command in the pipe (`cmd1 | cmd2`). If `cmd1` fails but `cmd2` succeeds, `$?` is 0. To get the status of all commands in the pipe, use the array `${PIPESTATUS[@]}` (Bash specific).

**Q: What does exit code 127 mean?**
**A:** Command not found.

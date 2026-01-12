---
---

## Summary
In Unix, standard output (`stdout`, descriptor 1) and standard error (`stderr`, descriptor 2) are separate streams. By default, both print to the terminal. To separate logs from errors or silence errors, you must use redirection operators.

## Detailed Explanation

### Redirection Syntax
*   `command > file`: Redirect stdout to file.
*   `command 2> file`: Redirect **stderr** to file.
*   `command &> file`: Redirect **both** to file (Bash shorthand).
*   `command > file 2>&1`: Redirect stdout to file, then point stderr to where stdout is going (File).

### Use Cases
*   **Silence Errors**: `ls /missing 2> /dev/null`.
*   **Separate Logs**: `app > app.log 2> app.error`.
*   **Combine**: `cron_job > cron.log 2>&1`.

## Go-Specific Context/Examples

Go programs write to `os.Stdout` (`fmt.Println`) and `os.Stderr` (`log.Println` uses stderr by default).

### Example: Writing to Stderr in Go
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Fprintln(os.Stdout, "This is stdout")
	fmt.Fprintln(os.Stderr, "This is stderr")
}
```
If you run `go run main.go > out.txt`, you will still see "This is stderr" on your screen, but "This is stdout" will be in the file.

## Interview Questions

**Q: Why use `2>&1`?**
**A:** It allows you to merge error messages into the standard output stream so they appear in chronological order in a single log file or pipe. `grep "error" script.log` works on both streams if they are merged.

**Q: What is `/dev/null`?**
**A:** The "bit bucket". It is a special device file that discards everything written to it (write succeeds, but data vanishes). Redirecting to it silences output.

**Q: How do you pipe `stderr` to `grep`?**
**A:** Pipes `|` only carry `stdout`. To grep errors, you must redirect stderr to stdout first: `command 2>&1 | grep "error"`.

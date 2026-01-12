---
---

## Summary
The `read` command is used to accept user input from the keyboard (Standard Input) or from a file descriptor and assign it to a variable. It is the primary mechanism for interactivity in Bash scripts.

## Detailed Explanation

### Basic Syntax
`read variable_name`
*   Waits for user to type and press Enter.
*   Stores input in `$variable_name`.
*   If no name is provided, stores in `$REPLY`.

### Common Options
*   **`-p "Prompt"`**: Display a prompt string before waiting.
    *   `read -p "Enter name: " name`
*   **`-s` (Silent)**: Do not echo input (for passwords).
    *   `read -s -p "Password: " pass`
*   **`-t N` (Timeout)**: Wait N seconds, then fail if no input.
*   **`-r` (Raw)**: Do not allow backslashes to escape characters. **Always use this.**

## Go-Specific Context/Examples

In Go, reading from Stdin is handled by `fmt.Scan` (simple) or `bufio.Scanner` (robust).

### Example: Reading input in Go
```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	reader := bufio.NewReader(os.Stdin)
	fmt.Print("Enter text: ")
	
	// Similar to 'read -p'
	text, _ := reader.ReadString('\n')
	fmt.Println("You entered:", text)
}
```

## Interview Questions

**Q: Why should you always use `read -r`?**
**A:** Without `-r`, `read` interprets backslashes as escape characters. If a user types `C:\Windows`, `read` might interpret `\W` or strip the slash. `-r` treats the input literally, which is almost always what you want for paths or user data.

**Q: How do you read a file line-by-line in Bash?**
**A:** Use a `while` loop with input redirection.
```bash
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```
Note: Setting `IFS=` prevents trimming leading/trailing whitespace.

**Q: How do you read multiple variables at once?**
**A:** `read var1 var2`. Bash splits the input by space (IFS). The first word goes to `var1`, the second to `var2`. If there are more words, the *rest* of the line goes to the last variable.

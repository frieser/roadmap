---
---

## Summary
While Ansible comes with thousands of modules, sometimes you need to perform a task that no existing module supports. You can write **Custom Modules** in any language (Python, Go, Bash, Ruby) that can parse JSON arguments and print JSON output.

## Detailed Explanation

### How Modules Work
1.  **Input**: Ansible takes the arguments from the playbook (`key=value`), creates a file with these args (JSON), and uploads it to the managed node.
2.  **Execution**: Ansible executes the module binary/script on the managed node, passing the argument file path.
3.  **Output**: The module does its work and prints a JSON object to **Standard Output (stdout)**.
    *   `{"changed": true, "msg": "Success"}`
    *   `{"failed": true, "msg": "Error description"}`

### Python vs Others
*   **Python**: Preferred because Ansible provides `AnsibleModule` helper classes to handle argument parsing, error handling, and file operations easily.
*   **Go/Binary**: Useful for performance or if you want to distribute a compiled, dependency-free binary module.

## Go-Specific Context/Examples

Writing an Ansible Module in Go is robust because it compiles to a single binary (no Python dependency issues on the target).

### Example: A "Hello World" Go Module

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os"
)

// Response structure expected by Ansible
type Response struct {
	Msg     string `json:"msg"`
	Changed bool   `json:"changed"`
	Failed  bool   `json:"failed"`
}

func main() {
	// 1. Read arguments file (path passed as $1)
	if len(os.Args) < 2 {
		fail("No argument file provided")
	}
	// argsFile := os.Args[1]
	// In a real module, read/parse this file for parameters

	// 2. Perform Logic
	// ... do something ...

	// 3. Output JSON
	resp := Response{
		Msg:     "Hello from Go!",
		Changed: true,
		Failed:  false,
	}
	printOutput(resp)
}

func printOutput(resp Response) {
	b, _ := json.Marshal(resp)
	fmt.Println(string(b))
}

func fail(msg string) {
	printOutput(Response{Msg: msg, Failed: true})
	os.Exit(1)
}
```

### Installation
Place the compiled binary in `library/my_go_module` relative to your playbook.

## Interview Questions

**Q: Why might you choose Go over Python for a custom module?**
**A:** Speed and Dependencies. If the managed node has a broken Python environment or lacks specific libraries (like a database driver), a static Go binary works out of the box. It also runs faster for compute-heavy tasks.

**Q: What are the two required return fields in the JSON output?**
**A:** Technically none are strictly required for *all* cases, but `changed` (boolean) is essential for idempotency reporting, and `failed` (boolean) is used to signal errors. `msg` is standard for logging.

**Q: How do you debug a custom module?**
**A:** Run ansible with `-vvv`. Ansible will keep the temporary module file on the remote server (instead of deleting it). You can then SSH to the server and run the module manually to see why it crashes or produces wrong JSON.

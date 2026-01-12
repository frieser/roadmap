---
---

## Summary
Tasks are the smallest units of action in Ansible. A task calls a **Module** with specific arguments to perform a single operation (e.g., "Install git"). Modules are the "tools" in the toolkit (binaries or scripts) that do the actual work.

## Detailed Explanation

### Anatomy of a Task
```yaml
- name: Descriptive Name (Logs show this)
  module_name:
    arg1: value
    arg2: value
  become: yes           # Optional: Run as sudo
  ignore_errors: yes    # Optional: Don't stop play if this fails
  register: result      # Optional: Save output to variable
```

### Common Task Keywords
*   **`become`**: Escalate privileges.
*   **`when`**: Conditional execution (`when: ansible_os_family == "Debian"`).
*   **`loop`**: Repeat task for a list of items.
*   **`notify`**: Trigger a handler if the task makes changes.

### Modules
Ansible ships with thousands of modules (`apt`, `copy`, `docker_container`, `git`, `user`). You can view documentation via `ansible-doc <module>`.

## Go-Specific Context/Examples

While standard modules are Python, you **can** write custom modules in Go. Ansible executes modules by pushing the binary to the node and running it.

### Example: Concept of a Go Module
If you compile a Go program that accepts JSON arguments from a file (passed as argument 1) and prints JSON to stdout, Ansible can use it.

```go
// my_module.go (compiled to binary)
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	// 1. Read args (Ansible passes a file path)
	// 2. Do logic
	// 3. Print JSON response
	resp := map[string]interface{}{
		"changed": true,
		"msg":     "Hello from Go Module",
	}
	b, _ := json.Marshal(resp)
	fmt.Println(string(b))
}
```

## Interview Questions

**Q: What is `register` used for?**
**A:** It captures the output of a task into a variable. You can then use this variable in subsequent tasks (e.g., in a `when` clause or to debug output).
```yaml
- shell: cat /etc/os-release
  register: os_info
- debug:
    msg: "{{ os_info.stdout }}"
```

**Q: How do you continue a playbook even if a task fails?**
**A:** Add `ignore_errors: yes` to the task.

**Q: What is the difference between `command` and `shell` modules?**
**A:** `command` executes the binary directly (safer, no pipes/redirects). `shell` executes via `/bin/sh`, allowing pipes (`|`) and redirects (`>`) but opening security risks.

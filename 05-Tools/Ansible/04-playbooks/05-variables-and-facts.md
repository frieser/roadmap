---
---

## Summary
Variables store dynamic values (like package names or file paths) that can be reused throughout a playbook. Ansible also automatically gathers system information from managed nodes, known as **Facts** (e.g., IP address, OS version), which can be used as variables.

## Detailed Explanation

### Defining Variables
1.  **Playbook**:
    ```yaml
    vars:
      http_port: 80
    ```
2.  **Inventory**: `host_vars/` or `group_vars/` directories.
3.  **Command Line**: `-e "http_port=8080"` (Highest precedence).

### Ansible Facts
At the start of a play, Ansible runs the `setup` module to gather facts.
*   `ansible_os_family`: "Debian", "RedHat".
*   `ansible_processor_vcpus`: Number of cores.
*   `ansible_default_ipv4.address`: Primary IP.

Usage: `{{ ansible_os_family }}`.

### Magic Variables
Variables automatically defined by Ansible:
*   `hostvars`: Access variables of other hosts.
*   `groups`: List of all groups.
*   `inventory_hostname`: Name of the current host being processed.

## Go-Specific Context/Examples

You can run the `setup` module from Go to get system info as a JSON struct.

### Example: Parsing Ansible Facts in Go

```go
package main

import (
	"encoding/json"
	"fmt"
	"os/exec"
)

// Partial struct for Ansible output
type SetupOutput struct {
	AnsibleFacts struct {
		OSFamily string `json:"ansible_os_family"`
		MemTotal int    `json:"ansible_memtotal_mb"`
	} `json:"ansible_facts"`
}

func main() {
	// Run: ansible localhost -m setup
	cmd := exec.Command("ansible", "localhost", "-m", "setup", "--connection", "local")
	
	output, _ := cmd.Output()
	
	// Note: Ansible output often contains a header line "localhost | SUCCESS =>" 
	// which needs stripping before JSON parsing in a real app.
	
	fmt.Println("Raw Output (truncated):", string(output)[:100])
}
```

## Interview Questions

**Q: How do you disable fact gathering to speed up a playbook?**
**A:** Set `gather_facts: no` at the top of the play. This is useful if you don't need system info (e.g., just copying a file) and saves time on every host connection.

**Q: What is the difference between `host_vars` and `group_vars`?**
**A:** `host_vars` apply to a specific single host. `group_vars` apply to all hosts in a specific group. `host_vars` override `group_vars`.

**Q: How do you debug variables?**
**A:** Use the `debug` module.
```yaml
- debug:
    var: ansible_facts['os_family']
```

---
---

## Summary
Ansible Ad-Hoc commands are quick, one-line commands used to perform simple tasks on one or more managed nodes without writing a full playbook. They are perfect for quick checks, simple file transfers, or rebooting servers.

## Detailed Explanation

### Syntax
```bash
ansible <host-pattern> -m <module> -a "<arguments>" -i <inventory>
```
*   **<host-pattern>**: `all`, `webservers`, or specific IP.
*   **-m <module>**: The module to run (e.g., `ping`, `shell`, `copy`). Default is `command`.
*   **-a <args>**: Arguments for the module (e.g., "src=foo dest=bar").

### Use Cases
1.  **Connectivity Check**: `ansible all -m ping`
2.  **System Reboot**: `ansible dbservers -a "/sbin/reboot" -b` (`-b` = become/sudo).
3.  **Quick Info**: `ansible all -m shell -a "free -m"` (Check memory usage).

## Go-Specific Context/Examples

You can execute Ansible ad-hoc commands from Go using `os/exec`.

### Example: Checking Connectivity from Go

```go
package main

import (
	"fmt"
	"log"
	"os/exec"
)

func main() {
	// Equivalent to: ansible all -m ping
	cmd := exec.Command("ansible", "all", "-m", "ping", "--connection", "local") // local for testing
	
	output, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatalf("Command failed: %v\n%s", err, output)
	}
	
	fmt.Println("Ping Result:")
	fmt.Println(string(output))
}
```

## Interview Questions

**Q: When should you use Ad-Hoc commands vs Playbooks?**
**A:** Use **Ad-Hoc** for one-off, temporary tasks that you don't need to repeat or document (e.g., checking uptime, quick reboot). Use **Playbooks** for configuration management, deployment, and anything that needs to be repeatable, version-controlled, and idempotent.

**Q: What is the default module if `-m` is omitted?**
**A:** The `command` module. So `ansible all -a "ls -la"` is valid and runs the `ls` command.

**Q: How do you run an ad-hoc command with sudo privileges?**
**A:** Add the `-b` (become) flag. Optionally use `-K` to prompt for the sudo password if passwordless sudo isn't configured.

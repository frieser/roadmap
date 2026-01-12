---
---

## Summary
Writing your first playbook involves defining the desired state of your system using YAML. It typically follows a pattern: Target Hosts -> Define Privileges -> List Tasks. The command `ansible-playbook` executes this plan.

## Detailed Explanation

### Step-by-Step Example
Goal: Install Nginx and ensure it is running.

**File: `site.yml`**
```yaml
---
- name: Setup Web Server
  hosts: all
  become: yes  # Run as root (sudo)

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

### Running It
```bash
ansible-playbook -i inventory.ini site.yml
```

### Output Colors
*   **Green**: No change needed (Idempotent).
*   **Yellow**: Changed (Action taken).
*   **Red**: Failed.

## Go-Specific Context/Examples

You can wrap this execution in a Go CLI tool to create a "One-Click Deploy" utility.

### Example: Go Wrapper

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

func main() {
	fmt.Println("Starting Deployment...")

	cmd := exec.Command("ansible-playbook", "-i", "hosts", "site.yml")
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("Deployment Failed!")
		os.Exit(1)
	}

	fmt.Println("Deployment Complete!")
}
```

## Interview Questions

**Q: What happens if I run the playbook twice?**
**A:** Ideally, nothing. Ansible is **idempotent**. The first time, it installs Nginx (Yellow/Changed). The second time, it sees Nginx is already present and started, so it does nothing (Green/Ok).

**Q: How do you dry-run a playbook?**
**A:** Use the `--check` flag (Check Mode). `ansible-playbook site.yml --check`. It simulates changes without applying them, though not all modules support this perfectly.

**Q: What does `become: yes` do?**
**A:** It tells Ansible to escalate privileges (usually via `sudo`) before executing the task. This is required for tasks like installing packages (`apt`) or modifying system services (`service`) that require root access.

---
---

## Summary
The Ansible Inventory defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. It can be a simple static file (`/etc/ansible/hosts`) or a dynamic script that queries a cloud provider.

## Detailed Explanation

### Formats
1.  **INI**: The classic format. Simple and readable.
    ```ini
    [webservers]
    web1.example.com
    web2.example.com port=80  # Inline variable
    ```
2.  **YAML**: Structured, supports complex variables better.
    ```yaml
    all:
      children:
        webservers:
          hosts:
            web1.example.com:
            web2.example.com:
              port: 80
    ```

### Grouping
*   **Groups**: Logical collections (`db_servers`, `us_east`).
*   **Children**: Groups of groups (`[production:children]` contains `webservers` and `db`).

### Variables
*   **Host Vars**: Specific to one machine (`ansible_host=10.0.0.1`).
*   **Group Vars**: Apply to all members (`ansible_user=admin`).

## Go-Specific Context/Examples

When building internal developer platforms in Go, you often need to generate inventory files for Ansible to use.

### Example: Generating JSON Inventory from Go
Ansible accepts JSON as a dynamic inventory format.

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	// Structure required by Ansible
	inventory := map[string]interface{}{
		"webservers": map[string]interface{}{
			"hosts": []string{"192.168.1.50", "192.168.1.51"},
			"vars": map[string]string{
				"http_port": "8080",
			},
		},
		"_meta": map[string]interface{}{
			"hostvars": map[string]interface{}{},
		},
	}

	b, _ := json.MarshalIndent(inventory, "", "  ")
	fmt.Println(string(b))
}
```

## Interview Questions

**Q: What is the `localhost` exception?**
**A:** By default, `localhost` does not need to be defined in the inventory if you run a playbook with `connection: local`. However, if you want to apply group vars to it, you should add it explicitly.

**Q: How do you check which hosts are in a group?**
**A:** `ansible webservers --list-hosts`.

**Q: What is the default location for inventory?**
**A:** `/etc/ansible/hosts`. This is often overridden by `-i inventory.ini` in the command line or `inventory = ./hosts` in `ansible.cfg`.

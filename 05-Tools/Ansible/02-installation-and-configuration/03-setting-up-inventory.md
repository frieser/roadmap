---
---

## Summary
An Ansible Inventory is a file (or script) that lists the managed nodes (hosts) you want to automate. It groups hosts for easier targeting (e.g., "webservers", "databases") and allows assigning variables specific to those hosts.

## Detailed Explanation

### Static Inventory
Simple text files, usually in INI or YAML format.

**INI Format (Standard)**
```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

[production:children]
webservers
dbservers
```

**YAML Format**
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
```

### Dynamic Inventory
A script (Python, Go, Binary) that queries an external source (AWS API, Terraform State, CMDB) and returns the inventory in JSON format. This is essential for cloud environments where IPs change frequently.

### Host Variables
You can define variables inline:
`web1.example.com http_port=8080`
Or in `group_vars/` and `host_vars/` directories (Best Practice).

## Go-Specific Context/Examples

You can write a Dynamic Inventory script in Go. Ansible expects the executable to accept `--list` and return JSON.

### Example: Simple Go Dynamic Inventory

```go
package main

import (
	"encoding/json"
	"flag"
	"fmt"
	"os"
)

type Group struct {
	Hosts []string          `json:"hosts"`
	Vars  map[string]string `json:"vars,omitempty"`
}

type Inventory map[string]interface{}

func main() {
	listFlag := flag.Bool("list", false, "List all hosts")
	flag.Parse()

	if *listFlag {
		inv := Inventory{
			"webservers": Group{
				Hosts: []string{"192.168.1.10", "192.168.1.11"},
				Vars:  map[string]string{"http_port": "80"},
			},
			"_meta": map[string]interface{}{
				"hostvars": map[string]interface{}{},
			},
		}

		enc := json.NewEncoder(os.Stdout)
		enc.Encode(inv)
	}
}
```
*Compile this to a binary `my_inventory` and run `ansible-playbook -i ./my_inventory site.yml`.*

## Interview Questions

**Q: What is the `all` group?**
**A:** It is the default group that contains every host listed in the inventory. You can use it to apply base configurations (like creating a common user) to every server.

**Q: How does Ansible determine which host variable to use if it's defined in multiple places?**
**A:** Variable Precedence. Generally: Extra Vars (`-e`) > Host Vars > Group Vars > Inventory Vars > Defaults. A variable defined for a specific host overrides a variable defined for the group.

**Q: Can a host belong to multiple groups?**
**A:** Yes. A server can be in `[webservers]`, `[ubuntu]`, and `[us-east]` simultaneously. It inherits variables from all of them (merging strategies apply).

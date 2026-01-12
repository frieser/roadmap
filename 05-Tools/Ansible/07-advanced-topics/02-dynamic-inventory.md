---
---

## Summary
Dynamic Inventory allows Ansible to query external data sources (like AWS, Azure, VMware, or a custom CMDB) to get the list of managed nodes in real-time. This is essential for cloud environments where servers are ephemeral (autoscaling) and IP addresses change constantly.

## Detailed Explanation

### The Problem with Static Inventory
In `hosts.ini`, you hardcode IPs. In AWS, if an Auto Scaling Group adds 5 new servers, `hosts.ini` is outdated instantly.

### How Dynamic Inventory Works
Instead of a text file, you point `-i` to an executable script or a plugin configuration.
1.  **Inventory Scripts (Old)**: Executables (Python/Go) that output JSON.
2.  **Inventory Plugins (New)**: YAML configuration files (e.g., `aws_ec2.yml`) that use Ansible's internal plugin system.

### JSON Format
The script must output JSON with groups and `_meta` (hostvars) to avoid N+1 queries.
```json
{
  "webservers": {
    "hosts": ["10.0.0.1", "10.0.0.2"]
  },
  "_meta": {
    "hostvars": {
      "10.0.0.1": {"region": "us-east-1"}
    }
  }
}
```

## Go-Specific Context/Examples

Writing a Dynamic Inventory script in Go is a great way to integrate Ansible with custom internal tools.

### Example: Go Inventory Script
A Go program that outputs the required JSON structure.

```go
package main

import (
	"encoding/json"
	"flag"
	"fmt"
	"os"
)

func main() {
	// Ansible passes --list
	listFlag := flag.Bool("list", false, "List hosts")
	flag.Parse()

	if *listFlag {
		// In reality, query your DB or Cloud API here
		inventory := map[string]interface{}{
			"app_group": map[string]interface{}{
				"hosts": []string{"192.168.1.50"},
			},
		}
		
		json.NewEncoder(os.Stdout).Encode(inventory)
	}
}
```
Compile to `my-inventory`. Run: `ansible-playbook -i ./my-inventory site.yml`.

## Interview Questions

**Q: Inventory Scripts vs Plugins: Which to use?**
**A:** **Plugins** are preferred in modern Ansible. They are maintained by the core team/vendors, perform better (no separate process spawning), and are easier to configure (YAML vs writing code). Scripts are deprecated but still useful for custom/legacy integrations.

**Q: How do you target specific AWS instances in dynamic inventory?**
**A:** You use **Keyed Groups**. In the `aws_ec2.yml` config, you can group hosts by tags, region, or availability zone.
```yaml
keyed_groups:
  - key: tags.Environment
    prefix: env
```
This creates groups like `env_production` or `env_staging` automatically.

**Q: What is the `_meta` key in the JSON output?**
**A:** It creates an optimization. Without `_meta`, Ansible calls the script with `--host <hostname>` for *every single host* to get variables (N+1 problem). With `_meta`, the script returns all variables for all hosts in one go, significantly speeding up the start of the play.

---
---

# Puppet

Puppet is one of the original configuration management tools, pioneering the declarative "Infrastructure as Code" approach. It is widely used in traditional enterprise IT for lifecycle management of servers.

## Summary

Puppet uses a **Declarative Domain Specific Language (DSL)**. Instead of describing *how* to do something (like a script), you describe *what* the state should be (e.g., "User 'deploy' should exist"). The Puppet Agent (Pull-based) figures out the steps to achieve that state.

## Detailed Explanation

### 1. Architecture: Agent-based Pull
*   **Puppet Master**: The central server controlling configuration information.
*   **Puppet Agent**: Runs on managed nodes. It sends facts (Facter) to the Master, receives a Catalog (compiled config), and enforces it.

### 2. The Catalog
When an agent connects, the Master compiles all relevant manifests (code) into a **Catalog**. This is a directed acyclic graph (DAG) of resources and dependencies. The agent applies this catalog.

### 3. Facter
A standalone tool that gathers system information (OS, IP, uptime) and sends it to the Master as variables.

---

## Go Implementation Example

Puppet is extensible via **External Facts**. While core facts are Ruby/C++, you can write an executable in **Go** that outputs JSON/YAML, and Facter will ingest it. This is useful for high-performance custom data gathering.

### Custom External Fact in Go
Compile this binary and place it in `/etc/facter/facts.d/my_fact`.

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
	"runtime"
)

// FactData represents the structure we want Puppet to see
type FactData struct {
	GoVersion    string `json:"go_version"`
	AppVersion   string `json:"app_version"`
	IsMaintenance bool   `json:"is_maintenance"`
}

func main() {
	// Gather data (Simulated)
	data := FactData{
		GoVersion:    runtime.Version(),
		AppVersion:   "2.4.1",
		IsMaintenance: false,
	}

	// Output JSON to stdout. Facter reads this.
	// Puppet can now use $facts['go_version'] in manifests.
	b, err := json.Marshal(data)
	if err != nil {
		os.Exit(1)
	}
	fmt.Println(string(b))
}
```

## Interview Questions

**Q: What is the difference between Declarative (Puppet) and Procedural (Chef/Scripting)?**
**A:**
*   **Declarative**: You define the *end state* ("Ensure Nginx is installed"). You don't care how it happens (yum vs apt). Puppet abstracts the implementation details.
*   **Procedural**: You write the *steps* ("Run `apt-get install nginx`"). You have explicit control over the order and execution logic.

**Q: What is Hiera?**
**A:** Hiera is Puppet's key-value lookup tool. It allows you to separate *data* from *code*. Instead of hardcoding "Nginx port = 80" in your manifest, you ask Hiera for `nginx_port`. Hiera can return different values based on the node's facts (e.g., dev vs prod, linux vs windows), enabling hierarchical configuration overrides.

**Q: How does Puppet handle dependencies?**
**A:** Puppet uses the `require` and `before` metaparameters to define ordering. Because manifests are compiled into a graph (DAG), resources are not necessarily applied in the order they are written in the file, but in the order defined by the dependency graph.

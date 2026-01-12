---
---

# PowerShell for DevOps

PowerShell is a cross-platform task automation and configuration management framework. Unlike traditional Unix shells that accept and return text, PowerShell accepts and returns **.NET objects**. This fundamental difference makes it incredibly powerful for managing structured data and complex systems.

## Summary

PowerShell (now **PowerShell Core** or `pwsh` on Linux/macOS) is essential for managing Windows environments, Azure resources, and increasingly cross-platform workflows. Its **Cmdlet** (Command-let) structure uses a standardized `Verb-Noun` syntax (e.g., `Get-Process`, `New-Item`). Its object-oriented pipeline allows properties to be accessed directly without complex text parsing (no need for `awk`/`sed`).

## Detailed Explanation

### 1. The Object Pipeline
In Bash, you pipe text. In PowerShell, you pipe objects.
*   **Bash**: `ps aux | grep chrome` (Returns text lines)
*   **PowerShell**: `Get-Process | Where-Object {$_.Name -eq "chrome"}` (Returns Process objects)
    *   You can then do: `... | Select-Object Id, CPU, WorkingSet`

### 2. Core Concepts
*   **Cmdlets**: Native PowerShell commands built into the shell (.NET classes).
*   **Modules**: Packages of cmdlets.
*   **Desired State Configuration (DSC)**: PowerShell's IaC platform for defining infrastructure as code.
*   **PSRemoting**: Execute commands on remote systems (similar to SSH, but object-aware).

### 3. Cross-Platform DevOps
With PowerShell Core (pwsh), you can write a single script that manages:
*   Active Directory on Windows
*   Filesystems on Linux
*   APIs via `Invoke-RestMethod` (which automatically parses JSON into objects)

---

## Go Implementation Example

While Go replaces many scripting needs, sometimes you need to invoke PowerShell to leverage specific Windows APIs or existing modules (like AzureAD or VMware PowerCLI).

### invoking PowerShell from Go
This example executes a PowerShell command to get process information, converts it to JSON within PowerShell, and then unmarshals it into a Go struct. This is a common pattern: **PowerShell for Data Gathering -> JSON -> Go for Logic**.

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os/exec"
)

// ProcessInfo represents the structure of the object returned by PowerShell
type ProcessInfo struct {
	Name string  `json:"Name"`
	Id   int     `json:"Id"`
	CPU  float64 `json:"CPU"`
}

func main() {
	// The PowerShell command:
	// 1. Get processes
	// 2. Filter for CPU usage > 0.1
	// 3. Select specific properties
	// 4. Convert to JSON for Go consumption
	psScript := `Get-Process | 
		Where-Object {$_.CPU -gt 0.1} | 
		Select-Object Name, Id, CPU | 
		ConvertTo-Json -Depth 1`

	// Execute 'pwsh' (PowerShell Core) with the command
	cmd := exec.Command("pwsh", "-NoProfile", "-Command", psScript)
	
	output, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatalf("PowerShell error: %v\nOutput: %s", err, string(output))
	}

	// Unmarshal JSON output into Go structs
	var processes []ProcessInfo
	// Note: ConvertTo-Json might return a single object or array. 
	// For robust code, you might need to handle both. 
	// This example assumes multiple processes (array).
	if err := json.Unmarshal(output, &processes); err != nil {
		// Fallback: Try unmarshalling a single object if array fails
		var p ProcessInfo
		if err2 := json.Unmarshal(output, &p); err2 == nil {
			processes = append(processes, p)
		} else {
			log.Fatalf("JSON Parse Error: %v", err)
		}
	}

	// Process data in Go
	fmt.Printf("Found %d active processes:\n", len(processes))
	for _, p := range processes {
		fmt.Printf("[%d] %s (CPU: %.2f)\n", p.Id, p.Name, p.CPU)
	}
}
```

## Interview Questions

**Q: What is the main difference between PowerShell and Bash pipelines?**
**A:** Bash passes **text streams** (strings) between commands, requiring tools like `cut` and `awk` to parse data. PowerShell passes **.NET objects**, preserving the data structure and types, allowing properties to be accessed directly (e.g., `$process.Id`) without parsing.

**Q: How does `Invoke-RestMethod` facilitate API interaction?**
**A:** `Invoke-RestMethod` sends HTTP requests and automatically converts the JSON or XML response into PowerShell objects. This eliminates the need for manual parsing tools like `jq`, making API interaction seamless in scripts.

**Q: What is Execution Policy in PowerShell?**
**A:** It is a safety feature (not a security boundary) that determines which scripts can run. Common settings are `Restricted` (no scripts), `RemoteSigned` (downloaded scripts must be signed), and `Bypass` (nothing is blocked). In CI/CD, you often use `-ExecutionPolicy Bypass`.

**Q: Can you run PowerShell on Linux?**
**A:** Yes, via **PowerShell Core (`pwsh`)**. It allows you to use the same object-oriented scripting language and many standard cmdlets on Linux, enabling cross-platform automation scripts.

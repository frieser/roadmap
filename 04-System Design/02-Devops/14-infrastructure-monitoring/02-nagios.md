---
---

# Nagios

Nagios is one of the oldest and most established monitoring systems (released in 1999). It focuses on alerting based on the status of hosts and services. While less modern than Prometheus, it is still widely used in legacy enterprise environments for its stability and simple "Green/Red" status model.

## Summary

Nagios uses a **Check-based** model. It runs plugins (executables) against hosts/services at regular intervals. These plugins return an exit code that determines the status: **OK (0)**, **WARNING (1)**, **CRITICAL (2)**, or **UNKNOWN (3)**. It excels at binary "Is it up?" checks rather than complex trend analysis.

## Detailed Explanation

### 1. Architecture
*   **Nagios Core**: The scheduler and event processor.
*   **Plugins**: Standalone scripts (Bash, Perl, Python, Go) that perform the actual checks (e.g., `check_http`, `check_disk`).
*   **NRPE (Nagios Remote Plugin Executor)**: An agent installed on remote servers to run plugins locally (e.g., check local disk usage) and report back to the Nagios server.

### 2. Configuration
Configuration is static and file-based (`.cfg` files). Adding a new host requires editing a file and reloading the daemon, which can be cumbersome in dynamic cloud environments compared to Prometheus's service discovery.

---

## Go Implementation Example

Writing a custom Nagios plugin in Go is simple: perform a check, print a status line to stdout, and exit with the correct code.

### Custom Disk Check Plugin
```go
package main

import (
	"fmt"
	"os"
	"syscall"
)

// Exit Codes
const (
	OK       = 0
	WARNING  = 1
	CRITICAL = 2
	UNKNOWN  = 3
)

func main() {
	path := "/"
	
	// Get disk usage
	var stat syscall.Statfs_t
	if err := syscall.Statfs(path, &stat); err != nil {
		fmt.Printf("DISK UNKNOWN - Could not stat %s: %v\n", path, err)
		os.Exit(UNKNOWN)
	}

	// Calculate usage
	total := stat.Blocks * uint64(stat.Bsize)
	free := stat.Bfree * uint64(stat.Bsize)
	used := total - free
	percent := (float64(used) / float64(total)) * 100

	// Thresholds
	msg := fmt.Sprintf("Disk usage at %.2f%% | used=%d;total=%d", percent, used, total)

	if percent > 90 {
		fmt.Printf("DISK CRITICAL - %s\n", msg)
		os.Exit(CRITICAL)
	} else if percent > 80 {
		fmt.Printf("DISK WARNING - %s\n", msg)
		os.Exit(WARNING)
	} else {
		fmt.Printf("DISK OK - %s\n", msg)
		os.Exit(OK)
	}
}
```

## Interview Questions

**Q: How does Nagios differ from Prometheus in terms of data storage?**
**A:**
*   **Nagios**: Primarily stores current state (Up/Down). Historical data is often limited or stored in RRD (Round Robin Database) files via addons like PNP4Nagios, which aggregates data (losing precision over time).
*   **Prometheus**: Stores all data as time-series. It keeps raw data points for a configurable retention period, allowing for high-precision historical analysis and ad-hoc querying.

**Q: What is a "Flapping" service in Nagios?**
**A:** Flapping occurs when a service changes state too frequently (e.g., OK -> CRITICAL -> OK -> CRITICAL) within a short period. This generates a storm of notifications. Nagios detects this statistical pattern and temporarily suppresses notifications for that service until it stabilizes.

**Q: What is a "Passive Check"?**
**A:** In a Passive Check, the external application initiates the check and sends the result to Nagios (via the command pipe). Nagios does not schedule or run the check itself; it just waits for the result. This is useful for monitoring asynchronous jobs (like backups) or for security reasons where the monitoring server cannot initiate connections to the client.

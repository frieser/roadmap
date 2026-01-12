---
---

# Monit

Monit is a small, open-source utility for managing and monitoring Unix systems. Unlike Prometheus or Nagios which are "Global" monitoring systems, Monit is typically a "Local" monitoring agent that can perform autonomous maintenance.

## Summary

Monit runs on the localhost. Its philosophy is: "If a service crashes, restart it immediately. If disk space is full, clean it." It is an active remediation tool, not just a passive observer. It uses a human-readable DSL in its configuration file (`monitrc`).

## Detailed Explanation

### 1. Capabilities
*   **Process Monitoring**: Checks if a PID file exists and the process is running. Can restart if it crashes.
*   **Resource Monitoring**: Checks CPU, Memory, Load Avg of specific processes.
*   **Filesystem**: Checks for file existence, checksum changes (security), or disk space.
*   **Network**: Connects to ports (TCP/UDP) to verify protocol responses (HTTP, SMTP).

### 2. Architecture
Single binary. No database. No central server (though M/Monit exists for aggregation). It runs as a daemon (`d`) and wakes up every `n` seconds to check all configured rules.

---

## Go Implementation Example

Monit is not extended via libraries. You integrate Go apps with Monit by ensuring your Go app behaves like a proper Unix daemon (PID file, signal handling) so Monit can control it.

However, you can write a Go program that generates a **Monit Status** report by querying Monit's embedded HTTP server XML API.

### Querying Monit Status from Go
```go
package main

import (
	"encoding/xml"
	"fmt"
	"log"
	"net/http"
)

// Simplified XML structure of Monit status
type Monit struct {
	Services []Service `xml:"service"`
}

type Service struct {
	Name   string `xml:"name"`
	Status int    `xml:"status"` // 0 = Running
}

func main() {
	// Monit exposes an XML API (usually on port 2812)
	url := "http://admin:monit@localhost:2812/_status?format=xml"

	resp, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	var m Monit
	if err := xml.NewDecoder(resp.Body).Decode(&m); err != nil {
		log.Fatal(err)
	}

	fmt.Println("Monit Services Status:")
	for _, s := range m.Services {
		state := "OK"
		if s.Status != 0 {
			state = "FAIL"
		}
		fmt.Printf("- %s: %s\n", s.Name, state)
	}
}
```

## Interview Questions

**Q: What is the primary use case for Monit in a modern Kubernetes world?**
**A:** In Kubernetes, the Kubelet handles restarting crashed containers (Liveness Probes). However, Monit is still useful for:
1.  **Non-Containerized Systems**: Legacy VMs where you need an automated watchdog to restart Nginx/MySQL if they crash.
2.  **Sidecar Process Management**: If you have a fat container running multiple processes (anti-pattern, but happens), Monit can manage them inside the container.

**Q: How does Monit detect a "Zombie" process vs a "Dead" process?**
**A:**
*   **Dead**: The PID file exists, but the process ID does not exist in the system process table. Monit sees this and restarts the service.
*   **Zombie**: The process exists but is unresponsive. Monit detects this via protocol checks (e.g., HTTP request times out) or resource checks (CPU usage is 0% or 100% for too long).

**Q: Explain the syntax `check process nginx with pidfile ...`.**
**A:** This is Monit DSL. It tells Monit to track the process ID stored in that file. If the ID changes (restart), Monit notices. If the ID vanishes, Monit triggers the `start program` command defined in the block.

---
---

## Summary
These tools allow you to inspect network interfaces, routing tables, and active connections. `netstat` and `ifconfig` are part of the deprecated `net-tools` package (though still widely used), while `ss` and `ip` are the modern replacements from the `iproute2` suite.

## Detailed Explanation

### Interfaces
*   **Old**: `ifconfig` (List IPs/MACs).
*   **New**: `ip addr show` (or `ip a`).

### Connections/Ports
*   **Old**: `netstat -tulpn`
    *   `-t` TCP, `-u` UDP, `-l` Listening, `-p` Process, `-n` Numeric (no DNS resolution).
*   **New**: `ss -tulpn`. Faster and shows more details.

### Routing
*   **Old**: `route -n`.
*   **New**: `ip route`.

## Go-Specific Context/Examples

Go's standard `net` package provides cross-platform access to this info.

### Example: Listing Interfaces in Go
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	ifaces, _ := net.Interfaces()
	for _, i := range ifaces {
		addrs, _ := i.Addrs()
		fmt.Printf("%s: %v\n", i.Name, addrs)
	}
}
```

## Interview Questions

**Q: Why is `netstat` considered deprecated?**
**A:** It relies on reading `/proc` files in a way that is slow when there are thousands of connections. `ss` uses the **Netlink** kernel API (socket-based), which is much faster and provides more detailed TCP state info.

**Q: How do you find which process is listening on port 8080?**
**A:** `sudo ss -lptn 'sport = :8080'` or `sudo netstat -nlp | grep 8080` or `lsof -i :8080`. Note that you need sudo to see the Process ID (PID) of processes you don't own.

**Q: What is `127.0.0.1`?**
**A:** The Loopback Address (localhost). Traffic sent here stays on the machine and does not touch the network card. `ifconfig` shows this as interface `lo`.

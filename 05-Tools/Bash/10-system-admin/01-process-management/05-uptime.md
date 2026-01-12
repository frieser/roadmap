---
---

## Summary
The `uptime` command gives a one-line summary of system status: the current time, how long the system has been running, how many users are logged in, and the system load averages for the past 1, 5, and 15 minutes.

## Detailed Explanation

### Output
`20:00:00 up 100 days, 2 users, load average: 0.05, 0.10, 0.15`

### Load Average Interpretation
*   **< Cores**: Healthy.
*   **= Cores**: At capacity.
*   **> Cores**: Overloaded (Latency increases).

A load of 1.0 on a single-core CPU means 100% utilization. On a quad-core CPU, 1.0 means 25% utilization.

## Go-Specific Context/Examples

You can retrieve uptime programmatically in Go using `syscall.Sysinfo`.

### Example: Getting Uptime in Go
```go
package main

import (
	"fmt"
	"syscall"
	"time"
)

func main() {
	info := syscall.Sysinfo_t{}
	syscall.Sysinfo(&info)
	
	uptime := time.Duration(info.Uptime) * time.Second
	fmt.Println("Uptime:", uptime)
}
```

## Interview Questions

**Q: Where does `uptime` get its data from?**
**A:** It reads from the `/proc` filesystem, specifically `/proc/uptime` and `/proc/loadavg` on Linux.

**Q: Is "Load Average" just CPU usage?**
**A:** No. In Linux, it includes processes waiting for **Disk I/O** (Uninterruptible Sleep state `D`). A system can have high load but low CPU usage if the disk is thrashing.

**Q: Why check 1, 5, and 15 minute averages?**
**A:** To identify trends.
*   1 min > 15 min: Load is increasing (Spike).
*   1 min < 15 min: Load is decreasing (Recovery).

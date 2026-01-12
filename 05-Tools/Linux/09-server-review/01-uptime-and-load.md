---
tags: ['linux', 'roadmap']
---

## Summary
The `uptime` command and its associated **Load Averages** provide a critical snapshot of system performance and resource saturation. By showing the average number of processes in the run queue over 1, 5, and 15-minute intervals, it allows administrators to distinguish between temporary spikes and sustained overloading, while also accounting for I/O bottlenecks.

## Detailed Explanation

### The `uptime` Command
The `uptime` utility is the simplest way to check how long a system has been active and its current load.

```bash
$ uptime
 14:20:05 up 35 days,  4:12,  2 users,  load average: 1.05, 0.70, 5.15
```

**Output Fields:**
- **Current Time**: `14:20:05`
- **Uptime**: `up 35 days, 4:12` (Since last boot)
- **Users**: `2 users` (Logged in via TTY/SSH)
- **Load Average**: `1.05, 0.70, 5.15` (1m, 5m, 15m averages)

### Interpreting Load Averages
In Linux, "Load" represents the number of processes in the following states:
1.  **Running**: Using the CPU.
2.  **Runnable**: Waiting for the CPU.
3.  **Uninterruptible Sleep**: Waiting for I/O (Disk, Network, etc.). This is a unique feature of Linux load averages compared to other Unix systems.

#### The CPU Core Relationship
A load average of `1.00` means exactly one process is either using or waiting for a CPU core. To understand if a system is overloaded, you must compare the load to the number of available **Logical CPU Cores**.

- **Load < Cores**: The system has "headroom". Processes are being handled immediately.
- **Load = Cores**: The system is at 100% capacity. No idle time, but no waiting yet.
- **Load > Cores**: The system is saturated. Processes are queuing up and waiting for execution or I/O.

**Example (4-Core CPU):**
- Load `2.00`: 50% utilization.
- Load `4.00`: 100% utilization.
- Load `8.00`: 200% utilization (Processes spend as much time waiting as they do running).

### Bash Implementation & Monitoring

#### Check CPU Cores
Before analyzing load, determine your core count:
```bash
# Get number of logical cores
nproc

# Get detailed CPU info
lscpu | grep "^CPU(s):"
```

#### Reading directly from the Kernel
The kernel exposes load averages via the `/proc` filesystem:
```bash
cat /proc/loadavg
# Example: 0.15 0.22 0.45 1/850 12345
```
The first three values are the 1, 5, and 15-minute averages. The fourth value shows `currently_running/total_processes`.

#### Real-time Monitoring
```bash
# Update every second
watch -n 1 uptime

# Alternative: 'top' or 'htop' (which also show load in the header)
top -b -n 1 | head -n 1
```

### Go Application
You can programmatically retrieve load averages in Go by reading `/proc/loadavg`.

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	// Load averages are stored in /proc/loadavg on Linux
	data, err := os.ReadFile("/proc/loadavg")
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}

	fields := strings.Fields(string(data))
	if len(fields) < 3 {
		fmt.Println("Error: unexpected /proc/loadavg format")
		return
	}

	fmt.Println("--- System Load ---")
	fmt.Printf("Last 1 minute:  %s\n", fields[0])
	fmt.Printf("Last 5 minutes: %s\n", fields[1])
	fmt.Printf("Last 15 minutes: %s\n", fields[2])
}
```

## Interview Questions

**Q: Why might a server have a high Load Average but low CPU utilization?**
**A:** This typically indicates an **I/O bottleneck**. Since Linux includes processes in "uninterruptible sleep" (waiting for disk or network I/O) in the load average calculation, a slow disk or hanging network mount can drive the load up even if the CPU is mostly idle.

**Q: If the 1-minute load is 10.0 and the 15-minute load is 2.0, what does this tell you?**
**A:** It indicates a **recent spike** in system activity. The load is rapidly increasing. If it were the other way around (15m=10.0, 1m=2.0), it would mean the system was recently overloaded but is now recovering.

**Q: How do you find the number of CPU cores to correctly interpret the load?**
**A:** You can use the `nproc` command, check `lscpu`, or count processors in `/proc/cpuinfo` (`grep -c ^processor /proc/cpuinfo`).

**Q: Is a load average of 2.0 "bad"?**
**A:** It depends entirely on the hardware. On a single-core Raspberry Pi, it's 200% load (bad). On a 32-core production server, it's less than 10% load (excellent).

---
---

## Summary
Performance monitoring is the practice of continuously tracking system resource utilization (CPU, Memory, Disk I/O, Network) to ensure health, identify bottlenecks, and optimize throughput. In a DevOps context, this involves using low-overhead CLI tools for real-time debugging and implementing automated metric collection for long-term observability. Key tools include `vmstat` for system-wide health, `iostat` for disk performance, and `sar` for historical analysis.

## Detailed Explanation

### 1. CPU Monitoring
CPU performance is measured by load average and utilization percentages across user, system, and idle states.

*   **`top` / `htop`**: Real-time process monitoring. `htop` provides a more user-friendly, colorized interface with visual bars for CPU/RAM.
*   **`vmstat` (Virtual Memory Statistics)**: 
    *   `vmstat 1 5`: Reports every 1 second, 5 times.
    *   **Key columns**:
        *   `r`: Processes waiting for run time (bottleneck if > CPU cores).
        *   `us`: User time.
        *   `sy`: System time (kernel overhead).
        *   `id`: Idle time.
        *   `wa`: I/O wait (high values indicate disk/network bottlenecks).
*   **`sar` (System Activity Reporter)**:
    *   `sar -u 1 3`: Reports CPU utilization.

### 2. Memory Monitoring
Tracking memory helps identify leaks and excessive swapping that degrades performance.

*   **`free`**: Displays total, used, and free memory.
    *   `free -h`: Human-readable format.
    *   **Note**: `buff/cache` is memory used by the kernel for disk caching but is reclaimable if applications need it.
*   **`vmstat`**:
    *   `swpd`: Virtual memory used.
    *   `si` / `so`: Swap in / Swap out. Non-zero values indicate memory pressure and thrashing.

### 3. Disk I/O Monitoring
Disk I/O bottlenecks often manifest as high CPU `iowait`.

*   **`iostat`**: Reports CPU and I/O statistics.
    *   `iostat -xz 1`: Extended statistics for active devices.
    *   **Key metrics**:
        *   `%util`: Percentage of time the disk was busy. Values > 80-90% indicate a bottleneck.
        *   `await`: Average time (ms) for I/O requests to be served.
        *   `svctm`: Service time (now deprecated, use `await`).
*   **`iotop`**: Like `top`, but for disk I/O usage per process.

### 4. Process and File Monitoring
*   **`lsof` (List Open Files)**:
    *   `lsof -i :80`: Find process using port 80.
    *   `lsof -p <PID>`: List all files opened by a specific process.
    *   `lsof /var/log/nginx/access.log`: See which process is writing to a log.
*   **`fuser`**: Identifies processes using files or sockets.

---

## Go Application: Metrics Collection

In modern infrastructure, metrics are collected programmatically. The Go ecosystem provides robust tools for this.

### Using `gopsutil`
`gopsutil` is the standard library for cross-platform system information.

```go
package main

import (
	"fmt"
	"time"

	"github.com/shirou/gopsutil/v3/cpu"
	"github.com/shirou/gopsutil/v3/mem"
)

func main() {
	// CPU Percent over 1 second
	percent, _ := cpu.Percent(time.Second, false)
	fmt.Printf("CPU Usage: %.2f%%\n", percent[0])

	// Memory Stats
	v, _ := mem.VirtualMemory()
	fmt.Printf("Total Memory: %v GB, Available: %v GB, Usage: %.2f%%\n", 
		v.Total/1024/1024/1024, v.Available/1024/1024/1024, v.UsedPercent)
}
```

### Prometheus Node Exporter Patterns
The Prometheus `node_exporter` is written in Go and uses the `procfs` package to read from `/proc` and `/sys`.

1.  **Direct Instrumenting**: Use `prometheus/client_golang` to expose custom metrics.
2.  **Custom Exporters**: Follow the `Collector` interface to gather metrics from proprietary sources.

```go
package main

import (
	"net/http"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	cpuTemp = prometheus.NewGauge(prometheus.GaugeOpts{
		Name: "cpu_temperature_celsius",
		Help: "Current temperature of the CPU.",
	})
)

func init() {
	prometheus.MustRegister(cpuTemp)
}

func main() {
	cpuTemp.Set(65.5) // Example value
	http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":8080", nil)
}
```

---

## Interview Questions

**Q: What is the difference between Load Average and CPU Utilization?**
**A:** CPU Utilization is the percentage of time the CPU was busy during a period. Load Average (shown in `uptime`) represents the average number of processes in the "runnable" state (using or waiting for CPU) and "uninterruptible" state (waiting for I/O). A load average higher than the number of CPU cores indicates a bottleneck.

**Q: You see high `%wa` (I/O Wait) in `vmstat`. What tools do you use next?**
**A:** High `iowait` means the CPU is idle because all runnable tasks are waiting for I/O. I would use `iostat -xz 1` to identify which disk is slow and `iotop` to find the specific process causing heavy disk activity.

**Q: How do you find which process is holding a deleted file's space?**
**A:** When a file is deleted but its space isn't reclaimed, it's usually because a process still has it open. Use `lsof | grep deleted` to find the PID and then restart the process or truncate the file descriptor via `/proc/<PID>/fd/`.

**Q: What do `si` and `so` columns in `vmstat` indicate?**
**A:** `si` (Swap In) and `so` (Swap Out) show memory moving between RAM and Disk. If these are consistently non-zero, the system is "thrashing," meaning it doesn't have enough physical RAM for its working set, leading to severe performance degradation.

---
---

## Summary
Process monitoring is a critical DevOps task involving the observation and management of running programs (processes) to ensure system health and performance. It encompasses identifying resource bottlenecks (CPU, memory, I/O), tracking process hierarchies, managing priorities, and terminating unresponsive or malicious tasks. Key Linux tools like `top`, `htop`, and `ps` provide the visibility needed to maintain highly available and performant infrastructure.

## Detailed Explanation

### 1. Key Linux Monitoring Tools

| Tool | Mode | Use Case | Key Features |
| :--- | :--- | :--- | :--- |
| **ps** | Snapshot | Scripting, reporting | Static list of processes. `ps aux` (BSD) or `ps -ef` (System V). |
| **top** | Real-time | Quick overview | Default dynamic monitor. Shows CPU, Memory, and Load Average. |
| **htop** | Interactive | Deep dive | User-friendly, scrollable, colorized. Allows killing/renicing processes directly. |
| **pstree** | Tree | Hierarchy | Visualizes parent-child relationships (e.g., systemd -> docker -> container). |
| **pgrep** | Search | Automation | Finds PIDs by name or pattern. |

### 2. Managing Process Priorities
Linux uses a **Nice Value** to determine process priority (range: -20 to 19).
* **Lower value (-20)**: Higher priority (less "nice" to others).
* **Higher value (19)**: Lower priority (very "nice").

* **nice**: Starts a process with a specific priority.
  ```bash
  nice -n 10 ./my-script.sh
  ```
* **renice**: Changes the priority of a *running* process.
  ```bash
  renice -n -5 -p 1234
  ```

### 3. Terminating Processes (Signals)
Processes are managed via **Signals**.
* **SIGTERM (15)**: Graceful termination. Allows the process to clean up (e.g., close DB connections).
* **SIGKILL (9)**: Forceful termination. Immediate stop, no cleanup.
* **SIGHUP (1)**: Hangup. Often used to reload configuration files.

### 4. Process Monitoring in Go (Golang)
DevOps engineers often build custom agents or monitoring tools using Go.

#### A. Basic Execution with `os/exec`
For simple tasks like running `ps` and reading its output.

```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	cmd := exec.Command("ps", "aux")
	output, err := cmd.Output()
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}
	fmt.Println(string(output))
}
```

#### B. Advanced Monitoring with `gopsutil`
`gopsutil` is the industry standard for cross-platform system stats.

```go
package main

import (
	"fmt"
	"time"

	"github.com/shirou/gopsutil/v3/process"
)

func main() {
	// List all processes
	pids, _ := process.Pids()

	for _, pid := range pids {
		p, err := process.NewProcess(pid)
		if err != nil {
			continue
		}

		name, _ := p.Name()
		cpu, _ := p.CPUPercent()
		mem, _ := p.MemoryPercent()

		if cpu > 10.0 { // Alert if CPU > 10%
			fmt.Printf("ALERT: Process [%s] (PID: %d) CPU Usage: %.2f%%\n", name, pid, cpu)
		}
		
		fmt.Printf("Process: %s | PID: %d | Mem: %.2f%%\n", name, pid, mem)
	}
}
```

#### C. Sending Signals with `os` and `syscall`
```go
package main

import (
	"os"
	"syscall"
)

func killProcess(pid int) error {
	proc, err := os.FindProcess(pid)
	if err != nil {
		return err
	}
	// Send SIGTERM
	return proc.Signal(syscall.SIGTERM)
}
```

## Interview Questions

**Q: What is the difference between `SIGTERM` and `SIGKILL`?**
**A:** `SIGTERM` (15) is a polite request for the process to terminate, allowing it to execute cleanup routines. `SIGKILL` (9) is an immediate termination by the kernel that cannot be caught or ignored by the process, potentially leaving resources in an inconsistent state.

**Q: What is a "Zombie Process" and how do you fix it?**
**A:** A zombie process is a process that has completed execution but still has an entry in the process table. This happens because the parent process hasn't yet read its exit status via `wait()`. To "fix" it, you usually need to kill the parent process or fix the parent's code to properly handle child termination.

**Q: How does the `load average` in `top` differ from CPU usage?**
**A:** CPU usage shows the percentage of time the CPU was active. Load average represents the average number of processes in a "runnable" or "uninterruptible" state (waiting for CPU or I/O). A load average higher than the number of CPU cores indicates the system is saturated.

**Q: How do you identify which process is using a specific port?**
**A:** Using `lsof -i :<port>` or `netstat -tulnp | grep <port>`. These tools map open files (including network sockets) to their respective PIDs.

**Q: What is the purpose of `nice` and `renice`?**
**A:** They are used to adjust process scheduling priority. `nice` is used when starting a new process, while `renice` modifies a process that is already running. This helps prevent background tasks (like backups) from starving interactive applications of CPU resources.

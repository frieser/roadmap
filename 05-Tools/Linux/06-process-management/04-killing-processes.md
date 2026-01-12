#Linux
---
tags: ['linux', 'roadmap', 'process-management']
---

## Summary
In Linux, managing process lifecycle often requires terminating unresponsive or unnecessary tasks. The primary tools for this are `kill`, `pkill`, and `killall`. These commands don't actually "kill" processes directly; instead, they send **signals** to them. The process then decides how to handle the signal (though some signals like `SIGKILL` cannot be ignored). Understanding the difference between terminating by PID versus by name, and the implications of different signal types, is crucial for system stability and data integrity.

## Detailed Explanation

### 1. The `kill` Command
The `kill` command is the most fundamental tool for sending signals to processes. It requires the **Process ID (PID)**.

**Basic Usage:**
```bash
# Send the default SIGTERM (15) to PID 1234
kill 1234

# Send a specific signal (SIGKILL) to PID 1234
kill -9 1234
# OR
kill -SIGKILL 1234
```

### 2. `pkill` and `killall`
These tools allow you to target processes by **name** rather than PID, which is often more convenient.

*   **`killall`**: Kills processes by their exact name.
*   **`pkill`**: Kills processes based on pattern matching (regex) and offers advanced filtering.

**Examples:**
```bash
# Kill all processes named 'firefox'
killall firefox

# Kill processes matching a pattern (e.g., any process with 'chrome' in the name)
pkill chrome

# Kill processes owned by a specific user
pkill -u username firefox

# Kill processes matching the full command line (not just the process name)
pkill -f "python script.py"
```

### 3. Understanding Linux Signals
Signals are software interrupts sent to a program to indicate an event.

| Signal | Name | Description |
| :--- | :--- | :--- |
| **1** | `SIGHUP` | Hangup. Often used to tell a daemon to reload its configuration. |
| **2** | `SIGINT` | Interrupt. Triggered by `Ctrl+C`. Graceful termination. |
| **9** | `SIGKILL` | Kill. Immediate termination by the kernel. Cannot be ignored or caught. |
| **15** | `SIGTERM` | Terminate. The default signal. Requests the process to exit gracefully. |
| **19** | `SIGSTOP` | Stop. Pauses the process. |
| **18** | `SIGCONT` | Continue. Resumes a stopped process. |

### 4. Safety and Best Practices
1.  **Always try `SIGTERM` first**: This allows the process to close file descriptors, finish database transactions, and delete temporary files.
2.  **Use `SIGKILL` as a last resort**: Forced termination can lead to data corruption or "stale" lock files.
3.  **Verify with `pgrep`**: Before using `pkill`, use `pgrep -l pattern` to see which processes would be affected.

## Go Application (Perspective)
In Go, interacting with process signals is common for implementing graceful shutdowns or managing child processes.

**Handling Signals in Go:**
```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	// Create a channel to receive signals
	sigs := make(chan os.Signal, 1)

	// Register the channel to receive specific signals
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

	fmt.Println("Waiting for signal...")
	sig := <-sigs // Block until a signal is received
	fmt.Printf("\nReceived signal: %s. Cleaning up...\n", sig)
}
```

**Sending Signals from Go:**
```go
func killProcess(pid int) error {
    process, _ := os.FindProcess(pid)
    return process.Signal(syscall.SIGTERM)
}
```

## Interview Questions

**Q: What is the difference between `SIGTERM` and `SIGKILL`?**
**A:** `SIGTERM` (15) is a request for termination that the process can catch, handle, or ignore. It is used for graceful shutdowns. `SIGKILL` (9) is handled by the kernel and terminates the process immediately without giving it a chance to clean up. It cannot be caught or ignored.

**Q: How do you kill a process if you only know its name?**
**A:** You can use `killall [name]` for an exact name match, or `pkill [pattern]` for a partial or regex match. Alternatively, you can find the PID using `pgrep [name]` or `ps aux | grep [name]` and then use `kill [PID]`.

**Q: Why might a process remain in the process table even after `kill -9`?**
**A:** This usually happens if the process has become a **Zombie** (`Z` state). A zombie is already dead, but its entry remains because the parent process hasn't acknowledged its exit status yet. Another reason is if the process is in an **Uninterruptible Sleep** (`D` state), usually waiting for I/O; the kernel won't deliver the signal until the I/O operation completes.

**Q: How can you send a signal to a process using `pkill` without killing it (e.g., sending `SIGHUP`)?**
**A:** Use the `--signal` or `-<signal>` flag: `pkill -HUP process_name` or `pkill --signal SIGHUP process_name`.

**Q: What command would you use to see all available signals on your system?**
**A:** The `kill -l` command lists all signals supported by the system.

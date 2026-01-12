---
---

## Summary
The `jobs` command lists the active jobs (processes started by the shell) running in the background or suspended. Each job is assigned a unique Job ID (e.g., `%1`, `%2`) which is different from the Process ID (PID).

## Detailed Explanation

### Job States
*   **Running (`&`)**: Executing in background.
*   **Stopped (`Ctrl+Z`)**: Suspended (paused) but still in memory.
*   **Terminated**: Finished or killed.

### Commands
*   `jobs`: List all jobs.
*   `jobs -l`: List with PIDs.
*   `kill %1`: Kill Job #1.

## Go-Specific Context/Examples

Go **Goroutines** are lightweight threads managed by the Go runtime, not the OS shell. You cannot use `jobs` to see goroutines. However, if a Go program spawns child processes (using `os/exec`), those are OS processes.

### Analogy
*   **Shell**: `script.sh &` -> Job 1.
*   **Go**: `go func() { ... }` -> Internal runtime scheduler.

## Interview Questions

**Q: How do you bring the most recent background job to the foreground?**
**A:** `fg` (without arguments) or `fg %-`.

**Q: What happens to background jobs when you close the terminal?**
**A:** They receive a `SIGHUP` (Hangup) signal and terminate. To keep them running, use `nohup command &` or `disown`.

**Q: Difference between PID and Job ID?**
**A:** PID is global to the OS kernel (e.g., 12345). Job ID is local to the current shell session (e.g., %1).

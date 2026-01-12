---
---

## Summary
`top` and `htop` are interactive process viewers. They provide a real-time view of system performance (CPU, Memory, Load) and the list of running processes. `htop` is a modern, colorful, scrollable alternative to the classic `top`.

## Detailed Explanation

### Top Header (The Dashboard)
1.  **Uptime**: How long system has been running.
2.  **Load Average**: 1, 5, 15 minute averages of the system load (runnable processes).
3.  **Tasks**: Running, Sleeping, Zombie.
4.  **CPU(s)**: `us` (User), `sy` (System/Kernel), `id` (Idle), `wa` (IO Wait).
5.  **Mem**: Total, Free, Used, Buff/Cache.

### Interactive Commands
*   `k`: Kill a process (enter PID).
*   `u`: Filter by User.
*   `P`: Sort by CPU.
*   `M`: Sort by Memory.

## Go-Specific Context/Examples

Go's runtime makes debugging easy. `gops` is a tool (like top) specifically for Go processes.

### Go Application Profiling
If a Go app consumes high CPU in `top`, you can take a **pprof** profile to see exactly which function is burning cycles.

## Interview Questions

**Q: What does a high "Load Average" mean?**
**A:** It represents the average number of processes waiting for CPU time or Disk I/O. If Load > Number of Cores, the system is overloaded (processes are queuing).

**Q: What is "IO Wait" (`wa`)?**
**A:** The percentage of time the CPU is idle *waiting* for disk I/O to complete. High `wa` means the bottleneck is the disk (slow DB, logging), not the CPU processing power.

**Q: What is a Zombie process?**
**A:** A process that has finished execution but its parent has not yet read its exit status (reaped it). It uses no memory/CPU but consumes a PID.

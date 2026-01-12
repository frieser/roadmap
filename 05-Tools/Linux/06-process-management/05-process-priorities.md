#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Process priority in Linux determines the amount of CPU time the kernel scheduler allocates to a process. It is primarily managed through **nice values** ranging from -20 (highest priority) to 19 (lowest priority). Tools like `nice` and `renice` allow users and administrators to influence process scheduling to ensure critical tasks receive adequate resources or to prevent background tasks from impacting system responsiveness.

## Detailed Explanation
### Nice Values and Priority
In Linux, every process has a **Priority (PR)** and a **Nice (NI)** value.
- **Nice Value (NI):** A user-space value used to influence the scheduler. Range: **-20 to 19**.
- **Priority (PR):** The actual priority used by the kernel. For normal processes, it is calculated as `PR = 20 + NI`.
- **Range:**
  - `-20`: Highest priority (least "nice" to other processes).
  - `0`: Default priority.
  - `19`: Lowest priority (very "nice", only runs when nothing else wants the CPU).

### The `nice` Command
Used to start a process with a specific priority.
- **Syntax:** `nice -n <value> <command>`
- **Example (Regular User):**
  ```bash
  nice -n 10 tar -czf backup.tar.gz /home/user
  ```
  *This starts the backup with a lower priority (10).*
- **Example (Root):**
  ```bash
  sudo nice -n -10 ./critical_service
  ```
  *This starts a service with a higher priority (-10).*

*Note: Regular users can only increase the nice value (0 to 19). Only root can set a negative nice value.*

### The `renice` Command
Used to change the priority of an already running process.
- **Syntax:** `renice -n <value> -p <pid>`
- **Example:**
  ```bash
  renice -n 5 -p 1234
  ```
- **Other targets:** `renice` can also target users (`-u`) or process groups (`-g`).
  ```bash
  renice -n 10 -u john
  ```
  *Lowers priority for all processes owned by user 'john'.*

### Monitoring Priorities
- **`top` / `htop`:** Displays `PR` and `NI` columns.
- **`ps`:** Use custom output formats to see priority info.
  ```bash
  ps -o pid,ni,pri,comm -p <pid>
  ```
  *Note: In `ps`, the `PRI` column might show a different calculation depending on the flags used (e.g., `PRI = 60 + NI` or `PRI = 80 - NI` in some older systems), but the concept remains the same.*

### Why use priorities?
1. **Background Tasks:** Run backups, indexers, or long-running scripts with `nice -n 19` so they don't lag the desktop or web server.
2. **Critical Services:** Ensure a real-time monitoring tool or a database has a slight edge over other processes.
3. **Resource Management:** Prevent a single runaway process from freezing the entire system.

## Interview Questions
1. **What is the range of nice values in Linux, and which value represents the highest priority?**
   - The range is **-20 to 19**. **-20** is the highest priority (it's the "least nice" value), while **19** is the lowest priority.

2. **How does the kernel calculate the priority (PR) of a normal process based on its nice (NI) value?**
   - For standard processes, the kernel priority is typically calculated as `PR = 20 + NI`. So, a process with NI 0 has a PR of 20, and a process with NI -20 has a PR of 0.

3. **Can a regular (non-root) user set a process to a nice value of -5?**
   - No. Regular users can only **increase** the nice value (making the process lower priority). Only the superuser (root) can decrease the nice value or set it to a negative number to increase priority.

4. **What is the difference between `nice` and `renice`?**
   - `nice` is used to set the priority of a process **at the time of its creation**, while `renice` is used to modify the priority of a process that is **already running**.

5. **If you have a CPU-intensive backup script running that is slowing down your web server, what command would you use to lower its priority?**
   - You would use `renice`. First, find the PID of the script, then run `renice -n 19 -p <PID>` to set it to the lowest possible priority.

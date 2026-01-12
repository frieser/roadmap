---
tags: ['linux', 'roadmap']
---

# Process Signals

## Summary
**Process Signals** are software interrupts sent to a running program to notify it of a specific event. They are a core mechanism in Linux for process control and Inter-Process Communication (IPC). Signals allow the kernel, other processes, or the user to interact with a running process by requesting termination, suspension, or custom actions.

## Detailed Explanation

### What are Signals?
A signal is a message sent to a process that interrupts its normal execution flow. When a process receives a signal, it can:
1. **Terminate**: The process stops running.
2. **Ignore**: The signal is discarded (not possible for all signals).
3. **Catch/Handle**: Execute a specific function (signal handler) in response.

### Common Signal Types

| Signal | Number | Name | Description | Handleable? |
| :--- | :--- | :--- | :--- | :--- |
| **SIGINT** | 2 | Interrupt | Triggered by `Ctrl+C`. Asks process to interrupt. | Yes |
| **SIGKILL** | 9 | Kill | Immediate termination by the kernel. | **No** |
| **SIGTERM** | 15 | Terminate | Default `kill` signal. Requests graceful shutdown. | Yes |
| **SIGSTOP** | 19 | Stop | Suspends the process (pauses execution). | **No** |
| **SIGCONT** | 18 | Continue | Resumes a process previously stopped by SIGSTOP. | Yes |
| **SIGHUP** | 1 | Hangup | Sent when a terminal is closed. Often used to reload configs. | Yes |

### Using the `kill` Command
The `kill` command is the primary tool for sending signals to processes. Despite its name, it can send any signal, not just "kill" signals.

```bash
# List all available signals
kill -l

# Send SIGTERM (graceful) to process PID 1234
kill 1234
kill -15 1234
kill -SIGTERM 1234

# Send SIGKILL (forced) to process PID 1234
kill -9 1234
```

### Signal Handling in Bash (`trap`)
Scripts can "catch" signals using the `trap` command to perform cleanup before exiting.

```bash
#!/bin/bash

# Define a cleanup function
cleanup() {
    echo "Caught SIGINT! Cleaning up temporary files..."
    rm -f /tmp/test_file
    exit 0
}

# Register the trap for SIGINT (Ctrl+C)
trap cleanup SIGINT

echo "Process is running (PID: $$). Press Ctrl+C to interrupt."
while true; do
    sleep 1
done
```

## Interview Questions

### 1. What is the difference between SIGTERM (15) and SIGKILL (9)?
**SIGTERM** is a polite request to terminate. It allows the process to catch the signal, close file descriptors, release locks, and exit gracefully. **SIGKILL** is an authoritative command from the kernel that kills the process immediately; it cannot be caught or ignored, meaning no cleanup happens.

### 2. Which signals cannot be caught or ignored?
The signals **SIGKILL (9)** and **SIGSTOP (19)** cannot be caught, blocked, or ignored. They are handled directly by the kernel to ensure the system administrator always has control over runaway processes.

### 3. What happens to a process when it receives SIGSTOP?
The process's execution is suspended by the kernel. It remains in memory and its state is preserved, but it does not receive any CPU time. It will remain in the "Stopped" state (indicated by `T` in `ps` or `top`) until it receives a **SIGCONT** signal.

### 4. How do you send a signal to multiple processes with the same name?
You can use `pkill` or `killall`. For example, `pkill -SIGTERM nginx` will send the termination signal to all processes named "nginx".

### 5. What is the significance of SIGHUP?
Originally, **SIGHUP** (Signal Hangup) was sent to a process when its controlling terminal was closed. In modern server environments, it is commonly used as a convention to tell a daemon (like Nginx or Apache) to reload its configuration files without restarting the entire process.

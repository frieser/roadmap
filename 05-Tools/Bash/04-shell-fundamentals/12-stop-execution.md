---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Stop Execution (Signals and Control)

## Summary

Bash provides several ways to stop, pause, and control command execution using keyboard signals and job control. Understanding `Ctrl+C`, `Ctrl+Z`, `Ctrl+D`, and related signals is essential for managing running processes, canceling commands, and handling stuck programs.

## Detailed Explanation

### Common Control Signals

| Shortcut | Signal | Effect |
|----------|--------|--------|
| `Ctrl+C` | SIGINT (2) | Interrupt - terminate foreground process |
| `Ctrl+Z` | SIGTSTP (20) | Suspend - pause foreground process |
| `Ctrl+D` | EOF | End of input - close stdin |
| `Ctrl+\` | SIGQUIT (3) | Quit - terminate with core dump |
| `Ctrl+S` | - | Stop terminal output (XOFF) |
| `Ctrl+Q` | - | Resume terminal output (XON) |

### Ctrl+C - Interrupt

```bash
# Stop a running command
$ sleep 100
^C                     # Sends SIGINT, command terminates

# Most programs handle SIGINT gracefully
$ python script.py
^C
KeyboardInterrupt      # Python catches and handles it

# Some programs ignore SIGINT
$ less file.txt        # Ctrl+C doesn't quit, use 'q'
```

### Ctrl+Z - Suspend

```bash
# Pause a running command
$ vim file.txt
^Z
[1]+  Stopped                 vim file.txt

# Process is still alive, just paused
jobs
# [1]+  Stopped                 vim file.txt

# Resume in foreground
fg %1

# Resume in background
bg %1
```

### Ctrl+D - End of File

```bash
# Signal end of input
$ cat > file.txt
Hello
World
^D                     # Ends input, closes file

# Exit shell (if line is empty)
$ bash
$ ^D                   # Exits nested shell

# Won't exit if there's text on line
$ exit^D               # Ctrl+D ignored, need Enter

# Disable Ctrl+D exit
set -o ignoreeof       # Require 'exit' to close shell
```

### Job Control

```bash
# Background a command
command &

# List jobs
jobs
# [1]   Running                 sleep 100 &
# [2]-  Stopped                 vim file.txt
# [3]+  Running                 python server.py &

# Bring to foreground
fg %2

# Send to background
bg %1

# Kill a job
kill %1

# Refer to jobs
%1                     # Job number 1
%vim                   # Job starting with 'vim'
%%                     # Current job
%+                     # Current job
%-                     # Previous job
```

### The kill Command

```bash
# Send signals to processes
kill PID              # Default: SIGTERM (15)
kill -9 PID           # SIGKILL - force kill
kill -SIGKILL PID     # Same as above

# Common signals
kill -2 PID           # SIGINT (like Ctrl+C)
kill -15 PID          # SIGTERM (graceful termination)
kill -9 PID           # SIGKILL (cannot be caught)
kill -19 PID          # SIGSTOP (pause)
kill -18 PID          # SIGCONT (resume)

# Kill by name
pkill nginx
pkill -9 nginx
killall python

# Kill all jobs
kill $(jobs -p)
```

### Trapping Signals in Scripts

```bash
#!/bin/bash

# Handle Ctrl+C gracefully
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/myapp.lock
    exit 1
}

trap cleanup SIGINT SIGTERM

# Your script logic
echo "Running... (Ctrl+C to stop)"
while true; do
    sleep 1
done
```

### When Commands Won't Stop

```bash
# If Ctrl+C doesn't work:
# 1. Try Ctrl+\  (SIGQUIT - stronger)
^\ 

# 2. Suspend and kill
^Z
kill %1

# 3. Kill from another terminal
ps aux | grep command
kill -9 PID

# 4. If terminal is frozen
# Ctrl+Q (resume output)
# Or open new terminal and kill
```

## Interview Questions

**Q: What is the difference between Ctrl+C and Ctrl+Z?**
**A:** `Ctrl+C` sends SIGINT to terminate the process. `Ctrl+Z` sends SIGTSTP to suspend (pause) it - the process stays in memory and can be resumed with `fg` or `bg`. Use Ctrl+Z to temporarily pause, Ctrl+C to stop completely.

**Q: What is the difference between SIGTERM and SIGKILL?**
**A:** SIGTERM (15) requests graceful termination - processes can catch it and clean up. SIGKILL (9) forces immediate termination - cannot be caught or ignored. Always try SIGTERM first; use SIGKILL only if the process won't respond.

**Q: What does Ctrl+D do?**
**A:** It sends EOF (End of File) to stdin, signaling no more input. For shells, on an empty line it's equivalent to typing `exit`. It doesn't send a signal - it closes the input stream.

**Q: How do you resume a suspended job?**
**A:** Use `fg` to bring it to the foreground, or `bg` to continue it in the background. Use `jobs` to see suspended processes and their job numbers, then `fg %N` or `bg %N` for a specific job.

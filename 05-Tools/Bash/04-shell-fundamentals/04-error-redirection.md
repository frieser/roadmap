---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Error Redirection

## Summary

Error redirection in Bash controls where standard error (stderr, fd 2) output goes. By default, both stdout and stderr go to the terminal, but they can be redirected independently. This is crucial for separating error messages from normal output, logging errors, or suppressing error noise in scripts.

## Detailed Explanation

### Basic Error Redirection

```bash
# Redirect stderr to file
ls /nonexistent 2> errors.txt

# Append stderr to file
ls /nonexistent 2>> errors.txt

# Redirect stderr to /dev/null (discard errors)
ls /nonexistent 2> /dev/null

# Show only errors (discard stdout)
ls /home /nonexistent > /dev/null
# Only error message appears
```

### Combining stdout and stderr

```bash
# Redirect both to same file
command > output.txt 2>&1

# Order matters! This is different:
command 2>&1 > output.txt   # stderr goes to terminal!

# Explanation:
# > output.txt      : stdout (1) goes to file
# 2>&1              : stderr (2) goes where stdout (1) goes (file)

# Bash 4+ shorthand
command &> output.txt       # Both stdout and stderr to file
command &>> output.txt      # Append both to file
```

### Redirecting stderr to stdout

```bash
# Send errors to the same place as stdout
command 2>&1

# Common pattern: capture both in variable
output=$(command 2>&1)

# Pipe both stdout and stderr
command 2>&1 | grep "pattern"

# Bash 4+ shorthand
command |& grep "pattern"
```

### Separate stdout and stderr

```bash
# stdout to one file, stderr to another
command > stdout.txt 2> stderr.txt

# Process stdout and stderr differently
command 2> >(while read line; do echo "[ERR] $line"; done) \
        1> >(while read line; do echo "[OUT] $line"; done)

# Practical: log errors separately
./script.sh > output.log 2> error.log
```

### Swapping stdout and stderr

```bash
# Advanced: swap stdout and stderr
command 3>&1 1>&2 2>&3
# 3>&1 : fd 3 = stdout
# 1>&2 : stdout = stderr  
# 2>&3 : stderr = fd 3 (original stdout)

# Now stdout goes to terminal as errors would
# and stderr goes to terminal as stdout would
```

### Error Handling Patterns

```bash
#!/bin/bash

# Redirect all script errors to log file
exec 2> /var/log/script_errors.log

# Function to log errors
log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $*" >&2
}

# Usage
if ! command -v required_tool &> /dev/null; then
    log_error "required_tool is not installed"
    exit 1
fi

# Capture stderr only
errors=$(command 2>&1 >/dev/null)
if [[ -n "$errors" ]]; then
    echo "Command had errors: $errors"
fi
```

### Practical Examples

```bash
# Silent command (no output at all)
command &> /dev/null

# Check if command succeeds quietly
if command &> /dev/null; then
    echo "Success"
else
    echo "Failed"
fi

# Log everything with timestamps
{
    echo "=== Script started $(date) ==="
    ./long_running_task.sh
    echo "=== Script ended $(date) ==="
} > script.log 2>&1

# Tee stderr for debugging
command 2> >(tee -a debug.log >&2)
```

## Interview Questions

**Q: What does `2>&1` mean?**
**A:** It redirects file descriptor 2 (stderr) to wherever file descriptor 1 (stdout) currently points. The `&` indicates it's a file descriptor, not a filename. Order matters: place it after the stdout redirection.

**Q: Why is `command 2>&1 > file` different from `command > file 2>&1`?**
**A:** Shell processes redirections left-to-right. In `2>&1 > file`: stderr goes to stdout (terminal), then stdout goes to file. In `> file 2>&1`: stdout goes to file, then stderr goes to stdout (file). The second captures both.

**Q: How do you discard both stdout and stderr?**
**A:** Use `command > /dev/null 2>&1` or the Bash shorthand `command &> /dev/null`. Both redirect all output to the null device.

**Q: How do you print errors to stderr in a script?**
**A:** Use `echo "error message" >&2`. This redirects echo's stdout to stderr. Convention: normal messages to stdout, error messages to stderr. This allows users to redirect them independently.

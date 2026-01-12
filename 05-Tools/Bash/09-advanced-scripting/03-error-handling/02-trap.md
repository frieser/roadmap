---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# trap (Signal Handling)

## Summary

The `trap` command catches signals and executes commands when they occur. It's essential for cleanup operations, graceful shutdowns, and handling interrupts (Ctrl+C). Proper use of trap ensures scripts clean up temporary files and release resources even when interrupted.

## Detailed Explanation

### Basic Syntax

```bash
trap 'commands' SIGNAL [SIGNAL...]

# Common signals
trap 'echo "Interrupted"' INT        # Ctrl+C
trap 'cleanup' EXIT                   # Script exit
trap 'echo "Terminated"' TERM        # kill command
trap 'handle_error' ERR              # Command failure (with set -e)
```

### EXIT Trap (Most Common)

```bash
#!/bin/bash
set -euo pipefail

TMPDIR=$(mktemp -d)

cleanup() {
    echo "Cleaning up..."
    rm -rf "$TMPDIR"
}

trap cleanup EXIT

# Your script logic
# Even if it fails, cleanup runs
echo "Working in $TMPDIR"
cp important.txt "$TMPDIR/"
process "$TMPDIR/important.txt"

# Cleanup automatically runs on exit
```

### Common Signals

| Signal | Number | Trigger |
|--------|--------|---------|
| EXIT | 0 | Script exit (always) |
| HUP | 1 | Terminal closed |
| INT | 2 | Ctrl+C |
| QUIT | 3 | Ctrl+\ |
| TERM | 15 | kill command |
| ERR | - | Command error |
| DEBUG | - | Before each command |
| RETURN | - | Function/source returns |

### Multiple Signals

```bash
#!/bin/bash

cleanup() {
    echo "Received signal, cleaning up..."
    # Cleanup code
    exit 1
}

# Trap multiple signals
trap cleanup INT TERM HUP

# Or handle differently
trap 'echo "Ctrl+C"' INT
trap 'echo "Killed"' TERM
trap 'cleanup' EXIT
```

### ERR Trap (with set -e)

```bash
#!/bin/bash
set -euo pipefail

handle_error() {
    local exit_code=$?
    local line_no=${BASH_LINENO[0]}
    echo "Error on line $line_no: exit code $exit_code" >&2
}

trap handle_error ERR

# Now errors include line numbers
false   # Error on line X: exit code 1
```

### Advanced: Stack Trace on Error

```bash
#!/bin/bash
set -euo pipefail

stacktrace() {
    local i=0
    echo "Stack trace:" >&2
    while caller $i; do
        ((i++))
    done 2>&1 | awk '{print "  " $3 ":" $1 " in " $2}' >&2
}

trap stacktrace ERR
```

### Reset and Remove Traps

```bash
# Remove trap
trap - EXIT

# Reset to default
trap - INT

# List current traps
trap -p

# Check specific trap
trap -p EXIT
```

### Lock File Pattern

```bash
#!/bin/bash
LOCKFILE=/var/run/myscript.lock

cleanup() {
    rm -f "$LOCKFILE"
}

# Check if already running
if [[ -f "$LOCKFILE" ]]; then
    echo "Already running" >&2
    exit 1
fi

# Create lock and ensure cleanup
echo $$ > "$LOCKFILE"
trap cleanup EXIT

# Your script logic here
sleep 60
```

### Practical Examples

```bash
# Graceful shutdown
trap 'echo "Shutting down..."; kill $(jobs -p) 2>/dev/null; exit' SIGINT SIGTERM

# Restore terminal settings
original_stty=$(stty -g)
trap 'stty "$original_stty"' EXIT

# Stop background processes
trap 'kill $(jobs -p) 2>/dev/null' EXIT
```

## Interview Questions

**Q: What is the purpose of `trap ... EXIT`?**
**A:** The EXIT trap runs when the script exits, regardless of how (success, failure, signal). It's perfect for cleanup: removing temp files, releasing locks, restoring settings.

**Q: How do you clean up temporary files even if a script is interrupted?**
**A:** Create a cleanup function and trap it on EXIT: `trap cleanup EXIT`. This ensures cleanup runs on normal exit, errors, and signals like Ctrl+C.

**Q: What is the difference between trapping INT and trapping EXIT?**
**A:** INT fires only on Ctrl+C (SIGINT). EXIT fires on any exit - normal completion, errors, or signals. For cleanup, EXIT is usually preferred as it covers all cases.

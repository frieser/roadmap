---
tags: ['bash', 'shell', 'linux', 'tools', 'roadmap']
---

# Pipes

## Summary

**Pipes** (`|`) connect the stdout of one command to the stdin of another, enabling data to flow through a chain of commands. This is the core of Unix philosophy: small, focused tools combined to perform complex tasks. Pipes enable powerful text processing, filtering, and transformation without intermediate files.

## Detailed Explanation

### Basic Pipe Usage

```bash
# command1 output → command2 input
ls -la | less

# Chain multiple commands
cat file.txt | grep "error" | sort | uniq -c | sort -rn

# Each | connects stdout → stdin
ps aux | grep nginx | awk '{print $2}' | xargs kill
```

### How Pipes Work

```mermaid
graph LR
    A[Command 1] -->|stdout| B[Pipe Buffer]
    B -->|stdin| C[Command 2]
    C -->|stdout| D[Pipe Buffer]
    D -->|stdin| E[Command 3]
```

```bash
# Commands run concurrently (not sequentially)
# Data flows through as produced

# Producer/consumer running together
tail -f /var/log/syslog | grep --line-buffered "error"
```

### Common Pipe Patterns

```bash
# Count occurrences
grep "ERROR" logfile.txt | wc -l

# Find top 10
ps aux | sort -k3 -rn | head -10  # Top 10 by CPU

# Search and transform
cat data.csv | cut -d',' -f2 | sort | uniq

# Filter and format
docker ps | grep "running" | awk '{print $1, $2}'

# Multi-step text processing
cat access.log | \
    grep "404" | \
    awk '{print $1}' | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -20
```

### Pipe vs Redirection

```bash
# Pipe: command → command
cat file.txt | grep "pattern"

# Redirection: command ← file or command → file
grep "pattern" < file.txt
grep "pattern" file.txt > results.txt

# Combined
grep "error" < input.txt | sort | uniq > output.txt
```

### Pipe with stderr

```bash
# Pipe only passes stdout; stderr goes to terminal

# To pipe both stdout and stderr:
command 2>&1 | another_command

# Bash 4+ shorthand
command |& another_command

# Pipe stderr only (advanced)
command 2>&1 >/dev/null | process_errors
```

### Named Pipes (FIFOs)

```bash
# Create named pipe
mkfifo mypipe

# Writer (in one terminal)
echo "Hello from writer" > mypipe

# Reader (in another terminal)
cat < mypipe    # Receives "Hello from writer"

# Named pipes persist in filesystem
ls -la mypipe   # prw-r--r-- (p = pipe)

# Clean up
rm mypipe
```

### PIPESTATUS Array

```bash
# Get exit status of each command in pipeline
false | true | false
echo "${PIPESTATUS[@]}"   # 1 0 1

# $? only gives last command's status
echo $?                    # 1

# Check if any command failed
set -o pipefail
false | true              # Whole pipeline fails
echo $?                   # 1
```

### xargs: Converting Pipe to Arguments

```bash
# Some commands don't read stdin
find . -name "*.txt" | rm     # WRONG: rm ignores stdin

# Use xargs to convert stdin to arguments
find . -name "*.txt" | xargs rm

# Handle filenames with spaces
find . -name "*.txt" -print0 | xargs -0 rm

# Limit arguments per invocation
echo {1..100} | xargs -n 10 echo
```

## Interview Questions

**Q: What is the Unix pipeline philosophy?**
**A:** Write small programs that do one thing well, produce output suitable for other programs, and can be connected together. Pipes enable this by connecting stdout of one program to stdin of another.

**Q: What is the difference between pipe and redirection?**
**A:** Pipes connect commands (stdout → stdin between processes). Redirection connects commands to files (stdin from file, stdout to file). Pipes are inter-process communication; redirections are file I/O.

**Q: How do you capture the exit status of all commands in a pipeline?**
**A:** Use the `PIPESTATUS` array in Bash: `${PIPESTATUS[0]}` is first command, `${PIPESTATUS[1]}` is second, etc. Enable `set -o pipefail` to make the pipeline's exit status reflect failures.

**Q: Why use `xargs` with pipes?**
**A:** Some commands (like `rm`, `mv`, `cp`) expect arguments, not stdin. `xargs` converts stdin lines into command arguments. Use `-0` with `find -print0` for safe handling of filenames with spaces.

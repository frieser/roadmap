---
tags: ['linux', 'roadmap']
---

## Summary
In Linux and Unix-like operating systems, **Standard Streams** are preconnected input and output communication channels between a computer program and its environment. These streams are identified by non-negative integers called **File Descriptors (FD)**. By default, there are three primary streams: `stdin` (0) for input, `stdout` (1) for normal output, and `stderr` (2) for error messages and diagnostics. This abstraction allows for powerful features like redirection and piping, enabling modular command execution.

## Detailed Explanation

### The Three Standard Streams

1.  **Standard Input (stdin) - File Descriptor 0**:
    *   The source from which a program reads its input data.
    *   Default source: Keyboard.
    *   Redirection symbol: `<`.
    
2.  **Standard Output (stdout) - File Descriptor 1**:
    *   The destination where a program writes its normal output data.
    *   Default destination: Terminal screen.
    *   Redirection symbol: `>` (overwrite) or `>>` (append).

3.  **Standard Error (stderr) - File Descriptor 2**:
    *   The destination for error messages or diagnostics.
    *   Kept separate from `stdout` so that errors can be seen even if normal output is redirected.
    *   Default destination: Terminal screen.
    *   Redirection symbol: `2>`.

### File Descriptors (FD)
A File Descriptor is an index into an entry in the kernel-resident data structure containing the details of all open files. When a process starts, it automatically opens three FDs: 0, 1, and 2.

### Redirection Examples (Bash)

#### Redirecting stdout to a file
```bash
# Overwrite file with output
ls -l > files.txt

# Append output to file
echo "New entry" >> files.txt
```

#### Redirecting stdin from a file
```bash
# Sort content of a file
sort < names.txt
```

#### Redirecting stderr
```bash
# Redirect only errors to a log file
ls /nonexistent 2> errors.log
```

#### Redirecting both stdout and stderr
```bash
# Modern Bash syntax to redirect both to the same file
command &> output.txt

# Older/Traditional syntax
command > output.txt 2>&1
```

#### Discarding output
```bash
# Send output to the "black hole"
command > /dev/null 2>&1
```

### Piping
Piping connects the `stdout` of one command to the `stdin` of another.
```bash
# Filter the list of files for "txt" files
ls | grep ".txt"
```

### Advanced: Custom File Descriptors
You can open additional file descriptors (3-9) using `exec`.
```bash
# Open file descriptor 3 for reading
exec 3< myfile.txt
# Read from it
read -u 3 line
echo $line
# Close it
exec 3<&-
```

## Interview Questions

**Q: Why do we have a separate stderr stream instead of just sending everything to stdout?**
**A:** Separating `stderr` allows users to redirect normal program output (e.g., to a file or another program) while still seeing error messages on the terminal. It prevents "polluting" the data stream with diagnostic messages, which is crucial for automation and piping.

**Q: How do you redirect both stdout and stderr to the same file in a single command?**
**A:** In modern Bash, you can use `&> filename`. In traditional POSIX shells, you use `> filename 2>&1`, which first redirects `stdout` to the file and then redirects `stderr` to the same location as `stdout`.

**Q: What is the difference between `>` and `>>`?**
**A:** `>` is the redirection operator for overwriting. It creates the file if it doesn't exist or truncates it to zero length if it does. `>>` is the append operator; it creates the file if it doesn't exist or adds the output to the end of the existing file.

**Q: What does `2>&1` mean?**
**A:** It means "redirect file descriptor 2 (stderr) to the same place as file descriptor 1 (stdout)". The `&` indicates that the following character is a file descriptor, not a filename.

**Q: How can you run a command and ensure it produces no output at all on the terminal?**
**A:** Redirect both stdout and stderr to `/dev/null`. For example: `command > /dev/null 2>&1` or `command &> /dev/null`.

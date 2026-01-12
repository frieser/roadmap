---
tags: ['linux', 'roadmap']
---

## Summary
A pipe (`|`) is a fundamental Linux operator used to redirect the standard output (`stdout`) of one command directly into the standard input (`stdin`) of another. This mechanism allows users to chain multiple small, specialized tools together to perform complex data processing tasks, embodying the Unix philosophy of "do one thing and do it well."

## Detailed Explanation

### How Pipes Work
In Linux, every process has three default communication channels: `stdin` (0), `stdout` (1), and `stderr` (2). A pipe connects the `stdout` of the first process to the `stdin` of the second process.

```bash
command1 | command2
```

The shell executes both commands simultaneously. As soon as `command1` produces output, it is buffered and made available to `command2`.

### Chaining Multiple Commands
You can chain any number of commands to create a "pipeline":

```bash
# Example: Find the 5 largest files in the current directory
ls -lh | sort -k 5 -h -r | head -n 5
```

### Piping Standard Error (stderr)
By default, only `stdout` is piped. To pipe `stderr` as well, you must redirect it to `stdout` first:

```bash
# Traditional way
command 2>&1 | next_command

# Bash 4+ shortcut
command |& next_command
```

### Pipes vs. xargs
A common point of confusion is when to use a simple pipe versus `xargs`.
- **Pipe (`|`)**: Sends text as `stdin`. Use it if the next command is designed to read from input (e.g., `grep`, `sort`).
- **xargs**: Converts `stdin` into **command-line arguments**. Use it if the next command expects arguments (e.g., `rm`, `cp`, `mkdir`).

```bash
# Correct: rm expects arguments
find . -name "*.tmp" | xargs rm

# Incorrect: rm will try to read from stdin (and usually fail/ignore it)
find . -name "*.tmp" | rm 
```

### Useful Tools for Pipelines
- **`tee`**: Reads from `stdin` and writes to both `stdout` and one or more files (effectively a "T-splitter" for pipes).
- **`pv`**: (Pipe Viewer) Monitors the progress of data through a pipe.

### Checking Pipe Status
In Bash, the exit status of a pipeline is normally the exit status of the **last** command. To see the status of all commands in the pipe, use the `${PIPESTATUS}` array.

```bash
ls /nonexistent | grep "test"
echo ${PIPESTATUS[0]} # Will show 2 (ls failed)
echo ${PIPESTATUS[1]} # Will show 1 (grep found nothing)
```

## Interview Questions

**Q: What is the primary difference between a pipe (`|`) and a redirect (`>`)?**
**A:** A pipe connects the output of one command to the input of another command (`cmd1 | cmd2`), whereas a redirect sends the output of a command to a file (`cmd1 > file.txt`).

**Q: How do you pipe only the standard error (stderr) to another command?**
**A:** You can redirect `stderr` to `stdout` and "silence" the original `stdout` by sending it to `/dev/null`: `command 2>&1 1>/dev/null | next_command`.

**Q: Why would you use `xargs` instead of a simple pipe?**
**A:** `xargs` is necessary when the receiving command does not accept input via `stdin` but instead requires command-line arguments. For example, `rm` doesn't read file names from `stdin`; it expects them as arguments, so we use `echo "file.txt" | xargs rm`.

**Q: How can you split a pipe's output so it goes to both a file and the next command?**
**A:** Use the `tee` command. For example: `ls | tee files.txt | grep "search_term"`.

**Q: What happens if the first command in a pipe fails? Does the second one still run?**
**A:** Yes, the shell starts all commands in a pipeline simultaneously. The second command will run but will likely receive an EOF or a broken pipe signal if the first command terminates early without producing output.

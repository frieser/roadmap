#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Input/Output (I/O) redirection is a fundamental feature of Linux shells (like Bash) that allows users to control the flow of data between commands, files, and devices. By default, programs receive input from the keyboard (**stdin**) and send output to the terminal (**stdout** for normal data and **stderr** for error messages). Redirection operators enable capturing output into files, reading input from files, or discarding unwanted streams by routing them to special devices like `/dev/null`.

## Detailed Explanation
In Linux, every process is associated with three standard data streams, identified by **file descriptors (FD)**:
- **0 (stdin)**: Standard Input (keyboard by default)
- **1 (stdout)**: Standard Output (screen by default)
- **2 (stderr)**: Standard Error (screen by default)

### 1. Output Redirection (`>` and `>>`)
Output redirection allows you to save the results of a command to a file instead of displaying them on the screen.

- **Overwrite (`>`)**: Redirects stdout to a file. If the file exists, its contents are deleted (clobbered) before writing.
  ```bash
  ls -l > file_list.txt
  ```
- **Append (`>>`)**: Redirects stdout to a file, adding the output to the end of the file without deleting existing content.
  ```bash
  echo "Log entry: $(date)" >> activity.log
  ```

### 2. Input Redirection (`<`)
Input redirection tells a command to read its data from a file instead of the keyboard.

```bash
wc -l < data.txt
# The shell opens data.txt and feeds it to wc as stdin
```

### 3. Error Redirection (`2>`)
Since standard output and standard error are different streams, `>` only captures successful output. To capture errors, you must use the file descriptor `2`.

```bash
ls /root 2> access_denied.log
# Captures the error message if the user lacks permissions
```

### 4. Redirecting Both Output and Error (`&>`)
To capture everything (both stdout and stderr) into a single file, use the combined operator.

- **Modern Bash**: `&>`
  ```bash
  grep -r "pattern" /etc &> results.txt
  ```
- **POSIX/Legacy**: `> file 2>&1`
  ```bash
  grep -r "pattern" /etc > results.txt 2>&1
  ```

### 5. The Null Device (`/dev/null`)
The `/dev/null` file is a special device known as the "bit bucket." Anything written to it is discarded. It is frequently used to silence commands.

- **Silence errors only**:
  ```bash
  find / -name "secret.txt" 2> /dev/null
  ```
- **Silence all output**:
  ```bash
  systemctl status nginx &> /dev/null
  ```

## Interview Questions

**Q: What is the difference between `>` and `>>`?**
**A:** `>` redirects output and overwrites the target file if it already exists. `>>` redirects output and appends it to the end of the file, preserving its original content.

**Q: How do you redirect only the errors of a command to a file?**
**A:** Use the `2>` operator followed by the filename (e.g., `command 2> errors.log`). The `2` refers to the file descriptor for standard error.

**Q: What does `2>&1` signify in a command?**
**A:** It instructs the shell to redirect standard error (file descriptor 2) to the same destination as standard output (file descriptor 1).

**Q: How can you discard all output from a command, including errors?**
**A:** Redirect both stdout and stderr to `/dev/null` using `&> /dev/null` or `> /dev/null 2>&1`.

**Q: What is a Here Document (`<<`)?**
**A:** It is a form of input redirection that allows you to pass a multi-line string to a command directly from the shell or a script until a specific delimiter is reached.

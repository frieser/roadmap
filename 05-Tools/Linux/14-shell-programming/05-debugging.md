#Linux
---
tags: ['linux', 'roadmap', 'shell', 'debugging']
---

## Summary
Shell debugging is a critical skill for developing robust and maintainable scripts. It involves using built-in shell options to trace execution, handle errors automatically, and catch common mistakes early. Key tools include the `set` command options (`-x`, `-e`, `-u`, `pipefail`) for runtime debugging and `shellcheck` for static analysis and linting.

## Detailed Explanation

### 1. Runtime Tracing: `set -x` (xtrace)
The `set -x` option is the most common tool for debugging. When enabled, the shell prints each command and its arguments to standard error before executing them.

- **Purpose**: To see exactly what the shell is doing, including variable expansion and globbing.
- **Usage**:
  ```bash
  #!/bin/bash
  set -x
  NAME="Antigravity"
  echo "Hello, $NAME"
  # Output:
  # + NAME=Antigravity
  # + echo 'Hello, Antigravity'
  # Hello, Antigravity
  ```
- **Selective Debugging**: You can enable it for specific blocks using `set -x` to start and `set +x` to stop.

### 2. Error Handling: `set -e` (errexit)
By default, Bash scripts continue executing even if a command fails. `set -e` changes this behavior.

- **Purpose**: To exit the script immediately if any command returns a non-zero exit status.
- **Behavior**:
  ```bash
  #!/bin/bash
  set -e
  ls /nonexistent_folder
  echo "This will never be printed"
  ```
- **Caveat**: It doesn't catch errors in commands that are part of a test (e.g., in an `if` statement) or part of a pipeline (unless `pipefail` is set).

### 3. Unset Variables: `set -u` (nounset)
Typos in variable names are a frequent source of bugs. By default, Bash treats an unset variable as an empty string.

- **Purpose**: To treat unset variables as an error and exit immediately.
- **Example**:
  ```bash
  #!/bin/bash
  set -u
  GREETING="Hello"
  echo "$GREETNG" # Notice the typo
  # Output: bash: GREETNG: unbound variable
  ```

### 4. Pipeline Errors: `set -o pipefail`
In a pipeline, the exit status is usually determined by the last command.

- **Problem**: `false | true` returns an exit status of 0.
- **Solution**: `set -o pipefail` ensures that the pipeline returns the status of the last command to exit with a non-zero status.
- **Best Practice**: Use `set -euo pipefail` at the start of your scripts for "Strict Mode".

### 5. Static Analysis: `shellcheck`
`shellcheck` is an external tool that analyzes your script without running it.

- **Benefits**:
  - Detects syntax errors.
  - Points out "gotchas" (e.g., forgetting to quote variables).
  - Recommends best practices.
- **Installation**: `sudo apt install shellcheck` (on Debian/Ubuntu).
- **Usage**: `shellcheck myscript.sh`

## Interview Questions

**Q: What is the "Bash Strict Mode" and why is it recommended?**
**A:** Bash Strict Mode refers to starting a script with `set -euo pipefail`. It makes scripts more robust by exiting on errors (`-e`), failing on unset variables (`-u`), and ensuring pipelines correctly report failures (`pipefail`). This prevents "silent failures" that can lead to data corruption or unexpected behavior.

**Q: How do you debug a subshell in Bash?**
**A:** Since subshells inherit the environment but not all shell options, you might need to pass `-x` explicitly to the subshell or set it inside the parentheses: `( set -x; command1; command2 )`. Alternatively, you can export `SHELLOPTS` so that child shells inherit options like `xtrace`.

**Q: What is the difference between `set -x` and `set -v`?**
**A:** `set -v` (verbose) prints shell input lines exactly as they are read, before any expansion. `set -x` (xtrace) prints commands and arguments *after* expansion and *before* execution. `set -x` is usually more helpful for seeing what values were actually passed to a command.

**Q: How can you ignore an error from a specific command when `set -e` is enabled?**
**A:** You can append `|| true` to the command: `command_that_might_fail || true`. This forces the exit status of that line to be 0, allowing the script to continue.

**Q: Why should you quote variables in Bash, and how does `shellcheck` help with this?**
**A:** Unquoted variables undergo word splitting and globbing, which can cause bugs if the variable contains spaces or special characters. `shellcheck` identifies these instances and provides warnings (e.g., SC2086) suggesting to "Double quote to prevent globbing and word splitting."

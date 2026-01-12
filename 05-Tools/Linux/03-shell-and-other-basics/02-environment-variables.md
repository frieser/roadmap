#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Environment variables are dynamic-named values that can affect the way running processes will behave on a computer. They are part of the environment in which a process runs and are inherited by child processes, making them essential for system configuration, path management, and passing information to applications without hardcoding.

## Detailed Explanation

### 1. Shell vs. Environment Variables
*   **Shell Variables**: These are local to the current shell instance. If you start a sub-shell or run a script, these variables are not available to it.
*   **Environment Variables**: These are "exported" variables. They are available to the current shell and all of its child processes.

### 2. Core Commands
*   **`printenv`**: Lists all environment variables. You can also specify a variable name (e.g., `printenv PATH`).
*   **`env`**: Similar to `printenv`, but also used to run a command in a modified environment.
*   **`set`**: Lists all variables, including shell variables, environment variables, and shell functions.
*   **`export`**: Promotes a shell variable to an environment variable.
*   **`unset`**: Deletes a variable from the current session.

### 3. Common Environment Variables
| Variable | Description |
| :--- | :--- |
| `PATH` | A colon-separated list of directories where the system looks for executable files. |
| `HOME` | The current user's home directory. |
| `USER` | The name of the current logged-in user. |
| `SHELL` | The path to the current user's shell (e.g., `/bin/bash`). |
| `EDITOR` | The default text editor for system utilities (e.g., `vim`, `nano`). |

### 4. Setting Variables (Bash Examples)

#### Temporary Variables
```bash
# Setting a shell variable (local)
MY_PROJECT="/home/user/projects/app"

# Exporting it to the environment (global for child processes)
export MY_PROJECT

# Combined one-liner
export API_KEY="secret_value_123"
```

#### Modifying the PATH
This is the most common use case for environment variables.
```bash
# Append a new directory to the existing PATH
export PATH=$PATH:/home/user/.local/bin

# Prepend a directory (gives it priority)
export PATH=/usr/local/go/bin:$PATH
```

### 5. Persistence
Variables set in the terminal are lost when the session ends. To make them permanent, add them to configuration files:

*   **`~/.bashrc`**: The most common place for user-specific variables in interactive shells.
*   **`~/.bash_profile`**: Used for login shells. Usually sources `.bashrc`.
*   **`/etc/environment`**: A system-wide configuration file (not a script) for all users.
*   **`/etc/profile`**: A system-wide initialization script for all users.

To apply changes in `.bashrc` immediately without logging out:
```bash
source ~/.bashrc
```

## Interview Questions

### 1. What is the difference between `env`, `printenv`, and `set`?
`printenv` displays only environment variables. `env` displays environment variables and allows running a program in a custom environment. `set` is more comprehensive, displaying environment variables, shell variables, and shell functions.

### 2. How do you make a variable available to a script executed from your shell?
You must use the `export` command. If you simply define `VAR=value`, it is a shell variable and won't be visible to the child process (the script). `export VAR=value` makes it an environment variable.

### 3. If you add a directory to `PATH` in your terminal, will it be there after a reboot?
No. Variables set in a terminal session are temporary. To make it persistent, you must add the `export PATH=...` command to a shell initialization file like `~/.bashrc`.

### 4. What happens if you define a variable in both `~/.bashrc` and `/etc/environment`?
The user-specific configuration (`~/.bashrc`) usually takes precedence or overrides/appends to the system-wide configuration, depending on how the variables are loaded. For `PATH`, they are usually concatenated.

### 5. How can you run a specific command with a different environment variable without changing your current shell environment?
You can prefix the command with the variable assignment: `VAR_NAME=value command`. For example: `NODE_ENV=production npm start`.

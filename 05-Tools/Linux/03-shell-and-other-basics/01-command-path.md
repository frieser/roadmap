#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
The **Command PATH** is an environment variable (`$PATH`) that tells the Unix-like shell which directories to search for executable files when a command is entered. Instead of typing the full path to a program (e.g., `/usr/bin/ls`), the shell uses the list of directories in `$PATH` to find and execute it automatically. Understanding how to view, modify, and troubleshoot the PATH is essential for system administration and development.

## Detailed Explanation

### The $PATH Variable
The `$PATH` variable is a colon-separated list of absolute paths. When you type a command like `grep`, the shell searches each directory in this list from left to right until it finds an executable file with that name.

#### Viewing the PATH
You can view your current PATH using the `echo` or `printenv` command:

```bash
echo $PATH
# Example Output:
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### Locating Commands: `which`, `type`, and `whereis`

#### `which`
The `which` command shows the full path of the executable that would be run. It only looks for files in the directories listed in your `$PATH`.

```bash
which python3
# Output: /usr/bin/python3
```

#### `type`
The `type` command is more comprehensive than `which` because it identifies whether a command is a:
- **Shell builtin** (like `cd` or `echo`)
- **Alias** (like `ls='ls --color=auto'`)
- **Function**
- **External file**

Use the `-a` flag to see all occurrences of a command.

```bash
type -a ls
# Output:
# ls is aliased to `ls --color=auto'
# ls is /usr/bin/ls

type cd
# Output: cd is a shell builtin
```

#### `whereis`
The `whereis` command locates the binary, source, and manual page files for a command. It searches a broader set of standard system directories than `which`.

```bash
whereis bash
# Output: bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz
```

### Modifying the PATH

#### Temporary Change
To add a directory to the PATH for the current session:

```bash
export PATH=$PATH:/home/user/my_scripts
```
*Note: Always append or prepend to the existing PATH to avoid losing access to standard commands.*

#### Permanent Change
To make the change persistent, add the export command to your shell's configuration file:
- **Bash**: `~/.bashrc` or `~/.bash_profile`
- **Zsh**: `~/.zshrc`
- **System-wide**: `/etc/profile` or files in `/etc/profile.d/`

### Security Note
It is generally considered a security risk to add the current directory (`.`) to your PATH, especially at the beginning. An attacker could place a malicious executable named `ls` in a directory, and if you `cd` into it and run `ls`, you might execute the malicious script instead of the system command.

## Interview Questions

**Q: What happens if two directories in $PATH contain an executable with the same name?**
**A:** The shell executes the one found in the directory that appears first (leftmost) in the `$PATH` variable. This is why the order of directories in the PATH is important.

**Q: Why should you use `type` instead of `which` to find out what a command does?**
**A:** `which` only looks for executable files in the filesystem. It cannot identify shell builtins, aliases, or functions. `type` is a shell builtin that understands the shell's internal command resolution logic.

**Q: How can you run a command that is not in your $PATH?**
**A:** You must provide the absolute path (e.g., `/opt/myapp/bin/start`) or a relative path (e.g., `./script.sh` if it's in the current directory) to the executable.

**Q: How do you permanently add `/usr/local/go/bin` to your PATH for your user only?**
**A:** Add the line `export PATH=$PATH:/usr/local/go/bin` to your `~/.bashrc` (for Bash) or `~/.zshrc` (for Zsh) file, then reload the configuration with `source ~/.bashrc` or by restarting the terminal.

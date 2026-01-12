---
tags: ['linux', 'roadmap']
---

# Linux Basic Commands

## Summary
Navigating the Linux file system is a fundamental skill that involves understanding the hierarchical directory structure and using essential CLI tools. Commands like `pwd`, `ls`, and `cd` allow users to determine their current location, list contents, and move between directories. Additionally, built-in documentation tools such as `man` and `--help` provide necessary guidance on command usage and options, forming the bedrock of efficient terminal interaction.

## Detailed Explanation

### The Linux File System Hierarchy
Unlike Windows, Linux does not use drive letters (C:, D:). Instead, everything starts from the **Root Directory**, represented by a single forward slash `/`. All files, directories, and even hardware devices are attached as branches to this root.

### 1. `pwd` - Print Working Directory
To know exactly where you are in the file system, use `pwd`. It returns the absolute path from the root to your current location.

```bash
$ pwd
/home/user/documents
```

### 2. `ls` - List Directory Contents
The `ls` command shows the files and folders within a directory. It is often used with flags to provide more detail:
- `-l`: Long format (shows permissions, owner, size, and timestamp).
- `-a`: All files (includes hidden files starting with a dot `.`).
- `-h`: Human-readable sizes (e.g., 2K, 4M).

```bash
$ ls -lah
total 24K
drwxr-xr-x  2 user user 4.0K Jan 10 12:00 .
drwxr-xr-x 20 user user 4.0K Jan 10 11:30 ..
-rw-r--r--  1 user user  125 Jan 10 12:00 .hidden_config
-rw-r--r--  1 user user 5.2K Jan 10 11:55 notes.txt
```

### 3. `cd` - Change Directory
Used to navigate between directories. You can use absolute paths (starting from `/`) or relative paths.
- `cd ..`: Move up one level (parent directory).
- `cd ~`: Move to the user's home directory.
- `cd -`: Move to the previous directory you were in.

```bash
$ cd /var/log          # Absolute path
$ cd ../etc           # Relative path (up one, then into etc)
$ cd ~                # Go home
```

### 4. Getting Help: `man` and `--help`
Linux provides extensive documentation within the terminal:
- `man <command>`: Opens the manual page for a command (use `q` to quit).
- `<command> --help`: Displays a brief usage summary.

```bash
$ man ls
$ mkdir --help
```

### 5. Identifying Commands: `type` and `which`
- `type`: Tells you if a command is a built-in shell function, an alias, or an external binary.
- `which`: Shows the full path of the executable binary that will be run.

```bash
$ type cd
cd is a shell builtin

$ which python3
/usr/bin/python3
```

### 6. Housekeeping: `history` and `clear`
- `history`: Lists previous commands executed in the session.
- `clear`: Clears the terminal screen (shortcut: `Ctrl + L`).

```bash
$ history | tail -n 5
$ clear
```

## Interview Questions

**Q: What is the difference between an absolute path and a relative path?**
**A:** An absolute path starts from the root directory (`/`) and provides the complete path to a file or directory (e.g., `/home/user/docs`). A relative path starts from the current working directory and uses `.` (current) or `..` (parent) to navigate (e.g., `../pics`).

**Q: How can you list all files in a directory, including hidden ones, with their sizes in a readable format?**
**A:** Use the command `ls -lah`. The `-l` flag enables long listing, `-a` includes hidden files, and `-h` converts byte sizes into KB, MB, or GB.

**Q: What command would you use to find the location of a binary file in the system's PATH?**
**A:** The `which` command (e.g., `which bash`) is used to locate the executable file associated with a command.

**Q: How do you return to the previous directory you were working in without typing the full path?**
**A:** Use the command `cd -`. This toggles between the current and the previous working directory.

**Q: How can you search for a specific command you ran earlier in your session?**
**A:** You can use the `history` command, often combined with `grep` (e.g., `history | grep "docker"`) to find specific past commands.

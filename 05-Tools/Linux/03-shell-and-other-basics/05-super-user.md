---
tags: ['linux', 'roadmap']
---

# Super User (sudo, su, root user)

## Summary
The **Super User**, commonly known as the **root** user, is the administrative account in Linux with unrestricted access to all commands and files. Due to the inherent security risks of operating as root, tools like `sudo` and `su` are used to manage administrative privileges. `su` allows a user to switch identities entirely, while `sudo` enables authorized users to execute specific commands with elevated privileges, typically requiring their own password rather than the root password.

## Detailed Explanation

### The Root User
In Linux, the root user is the system administrator account that has a User ID (UID) of **0**. It bypasses all file permission checks and can perform any action on the system, including deleting critical system files or changing any user's password.
*   **Username**: `root`
*   **UID**: `0`
*   **Risk**: Any mistake made while logged in as root can lead to irreversible system damage or security breaches.

### `su` (Substitute User)
The `su` command is used to become another user during a login session. By default, running `su` without arguments attempts to switch to the root user.
*   **`su`**: Switches to root but keeps the current user's environment variables.
*   **`su -` (or `su -l`)**: Starts a "login shell." This switches to root and loads the root user's environment (PATH, home directory, etc.), which is the recommended way to switch users.
*   **Password**: Requires the password of the **target user** (e.g., the root password).

### `sudo` (SuperUser Do)
`sudo` is a powerful utility that allows a permitted user to execute a command as the superuser or another user, as specified by the security policy.
*   **Usage**: `sudo [command]`
*   **Password**: Typically requires the **current user's password** for authentication, not the root password.
*   **Advantages**:
    *   **Logging**: Every command executed via `sudo` is logged (usually in `/var/log/auth.log` or `/var/log/secure`).
    *   **Granular Control**: You can restrict which users can run which commands via the sudoers file.
    *   **No Root Password Sharing**: Users don't need to know the root password to perform admin tasks.

### The Sudoers File (`/etc/sudoers`)
The `/etc/sudoers` file controls who can run what via `sudo`. 
*   **`visudo`**: Always use the `visudo` command to edit this file. It performs syntax checking before saving, preventing you from accidentally locking everyone out of sudo privileges due to a typo.
*   **Syntax Example**: `alice ALL=(ALL:ALL) ALL` (Allows user 'alice' to run any command as any user on any host).

### Go Application: Checking for Root Privileges
In Go development, you might need to ensure your program is running with root privileges (e.g., for binding to low-numbered ports or raw sockets).

```go
package main

import (
	"fmt"
	"os"
	"os/user"
)

func main() {
	// Method 1: Check Effective User ID
	if os.Geteuid() != 0 {
		fmt.Println("Error: This program must be run as root (UID 0)")
		os.Exit(1)
	}

	// Method 2: Get Current User details
	currentUser, err := user.Current()
	if err != nil {
		fmt.Printf("Error getting user: %v\n", err)
		return
	}

	fmt.Printf("Running as superuser: %s (UID: %s)\n", currentUser.Username, currentUser.Uid)
}
```

## Interview Questions

**Q: What is the difference between `sudo` and `su`?**
**A:** `su` (substitute user) switches the entire shell session to another user (requiring the target user's password), whereas `sudo` (superuser do) runs a specific command with elevated privileges (typically requiring the current user's password). `sudo` is generally preferred for its logging capabilities and granular permission management.

**Q: Why is it considered a security risk to log in directly as the root user?**
**A:** Logging in as root violates the Principle of Least Privilege. Any command executed—including accidental ones or those triggered by malicious software—has full system access. Using `sudo` provides an audit trail and ensures that administrative powers are only invoked when specifically needed.

**Q: What does the `su -` command do that a simple `su` does not?**
**A:** `su -` starts a login shell, meaning it resets the environment variables (like `$PATH`, `$HOME`, and `$SHELL`) to match the target user's configuration. A plain `su` preserves the original user's environment, which can lead to issues if root-specific tools are not in the current `$PATH`.

**Q: How do you safely edit the `/etc/sudoers` file?**
**A:** You should always use the `visudo` command. It locks the file to prevent concurrent edits and, most importantly, validates the syntax before saving. If the syntax is invalid, it warns the user and refuses to save, preventing system-wide sudo failures.

**Q: What is the UID of the root user?**
**A:** The UID (User ID) of the root user is always **0**.

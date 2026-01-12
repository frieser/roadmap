---
tags: ['linux', 'roadmap', 'tools']
---

# File Permissions (chmod, chown, chgrp, octal/symbolic notation)

## Summary
Linux file permissions are a fundamental security layer that determines which users can read, write, or execute files and directories. This system categorizes access for the **Owner**, **Group**, and **Others** using a set of bits (`rwx`). Mastery of commands like `chmod`, `chown`, and `chgrp`, along with special bits like SUID, SGID, and the Sticky Bit, is essential for system administration and securing sensitive data.

## Detailed Explanation

### 1. Basic Permission Structure
Every file and directory in Linux has three sets of permissions:
- **User (u)**: The owner of the file.
- **Group (g)**: Users who are members of the file's group.
- **Others (o)**: All other users on the system.

Each set can have three types of access:
- **Read (r)**: View file content or list directory contents.
- **Write (w)**: Modify file content or add/remove files in a directory.
- **Execute (x)**: Run a file as a program or enter/traverse a directory.

### 2. Notation Types

#### Octal (Numeric) Notation
Permissions are represented by three digits, each ranging from 0 to 7.
- `4`: Read (r)
- `2`: Write (w)
- `1`: Execute (x)
- `0`: No permission

**Example calculation**: `rwx` = 4+2+1 = 7 | `rw-` = 4+2+0 = 6 | `r-x` = 4+0+1 = 5.

```bash
# Set owner to rwx, group and others to r-x (755)
chmod 755 script.sh

# Set owner to rw-, group and others to r-- (644)
chmod 644 config.txt
```

#### Symbolic Notation
Uses letters and operators (`+`, `-`, `=`) to modify permissions.
- `u` (user), `g` (group), `o` (others), `a` (all)

```bash
# Add execute permission to the owner
chmod u+x script.sh

# Remove write permission from group and others
chmod go-w sensitive_file.txt

# Set exact permissions for all (rwx for owner, r for others)
chmod u=rwx,g=rx,o=r data.csv
```

### 3. Ownership Commands

#### chown (Change Owner)
Changes the user and/or group ownership of a file.

```bash
# Change owner to 'alice'
chown alice document.pdf

# Change owner to 'alice' and group to 'devs'
chown alice:devs project_folder/

# Change ownership recursively
chown -R root:root /var/www/html
```

#### chgrp (Change Group)
Specifically changes the group ownership.

```bash
chgrp admins logs/
```

### 4. Special Permissions

#### SUID (Set User ID)
When set on an executable, it runs with the privileges of the file's **owner**.
- **Octal**: `4xxx` (e.g., `4755`)
- **Symbolic**: `u+s`

```bash
# Example: the 'passwd' command needs SUID to modify /etc/shadow
chmod u+s /usr/bin/passwd
```

#### SGID (Set Group ID)
- On files: Runs with the privileges of the file's **group**.
- On directories: New files created inside inherit the parent directory's group.
- **Octal**: `2xxx` (e.g., `2775`)
- **Symbolic**: `g+s`

```bash
# Ensure files in a shared folder belong to the 'team' group
chmod g+s /shared/project
```

#### Sticky Bit
On directories, it prevents users from deleting or renaming files they don't own, even if they have write permission on the directory.
- **Octal**: `1xxx` (e.g., `1777`)
- **Symbolic**: `+t`

```bash
# Standard for /tmp to prevent users from deleting each other's files
chmod +t /tmp
```

## Interview Questions

**Q: What is the difference between `chmod 777` and `chmod 1777`?**
**A:** `chmod 777` gives everyone full read, write, and execute permissions. `chmod 1777` does the same but adds the **Sticky Bit**, which ensures that only the file owner, the directory owner, or the root user can delete or rename files within that directory.

**Q: How do you change the owner and group of a directory and all its contents in a single command?**
**A:** Use the `chown` command with the `-R` (recursive) flag: `chown -R user:group directory_name`.

**Q: What happens if you have `r-x` permissions on a directory but no `w` permission?**
**A:** You can list the files in the directory (`r`) and enter the directory or access files inside if you know their names (`x`), but you cannot create, delete, or rename any files within that directory (`w`).

**Q: How does SGID behave differently when applied to a file versus a directory?**
**A:** On a file, SGID executes the file with the permissions of the file's group. On a directory, SGID ensures that any new files or subdirectories created within it inherit the group ownership of the parent directory, rather than the primary group of the user who created them.

**Q: In symbolic notation, how would you remove write and execute permissions for 'others' without affecting the owner or group?**
**A:** `chmod o-wx filename`.

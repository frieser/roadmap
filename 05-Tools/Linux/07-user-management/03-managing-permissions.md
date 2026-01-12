---
tags: ['linux', 'roadmap', 'tools']
---

# Managing Permissions (ACLs, getfacl, setfacl, visudo)

## Summary
Standard Linux permissions follow the `ugo` (User, Group, Others) model, which can be restrictive in multi-user environments. **Access Control Lists (ACLs)** extend this by allowing fine-grained permissions for specific users and groups. Parallel to this, managing administrative access via the **sudoers** configuration ensures that users can execute privileged commands securely. Mastering `getfacl`, `setfacl`, and `visudo` is essential for implementing the principle of least privilege in modern Linux systems.

## Detailed Explanation

### 1. Access Control Lists (ACLs)
ACLs allow you to grant permissions to multiple users or groups on a single file or directory, surpassing the traditional owner/group limitation.

#### Viewing and Setting ACLs
The two primary tools are `getfacl` (to view) and `setfacl` (to modify).

```bash
# View existing ACLs for a file
getfacl secret_report.txt

# Grant user 'john' read and write access
setfacl -m u:john:rw secret_report.txt

# Grant group 'auditors' read-only access
setfacl -m g:auditors:r secret_report.txt

# Remove an ACL entry for a specific user
setfacl -x u:john secret_report.txt

# Remove all extended ACLs (return to standard ugo)
setfacl -b secret_report.txt
```

#### Default ACLs
When set on a directory, **Default ACLs** ensure that all new files and subdirectories created within it inherit specific permissions.

```bash
# Set a default ACL for the 'developers' group on a shared directory
setfacl -d -m g:developers:rwx /opt/project_data
```

---

### 2. Sudoers and Administrative Access
The `sudo` command allows users to run programs with the security privileges of another user (usually root). This access is controlled by the `/etc/sudoers` file.

#### Using visudo
You should **never** edit `/etc/sudoers` with a standard text editor. Instead, use `visudo`. It locks the file against simultaneous edits and validates the syntax before saving, preventing accidental lockouts.

```bash
sudo visudo
```

#### Sudoers Syntax
The configuration follows the pattern: `User Host=(Runas_User:Runas_Group) Commands`.

```bash
# Give 'alice' permission to run all commands as any user
alice ALL=(ALL:ALL) ALL

# Give members of the 'wheel' group full sudo access
%wheel ALL=(ALL:ALL) ALL

# Allow 'bob' to run the system update command without a password
bob ALL=(ALL) NOPASSWD: /usr/bin/dnf update, /usr/bin/apt-get update

# Best practice: Use /etc/sudoers.d/ for modular configuration
# This keeps the main sudoers file clean and survives package updates.
sudo visudo -f /etc/sudoers.d/90-custom-apps
```

---

## Interview Questions

**Q: How can you tell if a file has an ACL applied using the `ls` command?**
**A:** When running `ls -l`, a file with an ACL will have a plus sign (`+`) at the end of the permission string (e.g., `-rw-rw-r--+`).

**Q: What is the difference between `setfacl -m` and `setfacl -x`?**
**A:** `-m` (modify) is used to add a new ACL entry or change an existing one, while `-x` (remove) is used to delete a specific entry for a user or group from the ACL.

**Q: Why is it recommended to use `/etc/sudoers.d/` instead of editing `/etc/sudoers` directly?**
**A:** Using `/etc/sudoers.d/` allows for modularity and easier management. It prevents the main configuration file from becoming cluttered and ensures that custom rules are not overwritten when the system's core `sudo` package is updated.

**Q: What does a "Default ACL" do when applied to a directory?**
**A:** A Default ACL does not affect the permissions of the directory itself, but it acts as a template for any new files or subdirectories created inside it, ensuring they automatically inherit the specified permissions.

**Q: What happens if you make a mistake in the `/etc/sudoers` file and don't use `visudo`?**
**A:** A syntax error can break the `sudo` command entirely. If you don't have another way to access the root account (like `su` or physical access), you may be unable to fix the error or perform any administrative tasks, effectively locking yourself out of the system's management.

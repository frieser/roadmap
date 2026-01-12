---
tags: ['linux', 'roadmap', 'tools']
---

# Users and Groups

## Summary
In Linux, user and group management is fundamental for system security and access control. This topic covers how to create and delete groups using `groupadd` and `groupdel`, and how to understand the configuration files `/etc/passwd` and `/etc/group` which store user and group definitions respectively.

## Detailed Explanation

### **Configuration Files**

#### **/etc/passwd**
This file stores essential information for each user account. Each line represents one user and contains seven fields separated by colons:
1. **Username**: Login name.
2. **Password**: Usually `x`, indicating the password is encrypted in `/etc/shadow`.
3. **UID**: User Identifier (0 for root).
4. **GID**: Primary Group Identifier.
5. **GECOS**: User information (full name, etc.).
6. **Home Directory**: Path to the user's home folder.
7. **Login Shell**: The shell that starts when the user logs in (e.g., `/bin/bash`).

#### **/etc/group**
This file defines the groups on the system and lists their members.
1. **Group Name**: Name of the group.
2. **Group Password**: Usually `x` (rarely used).
3. **GID**: Group Identifier.
4. **User List**: Comma-separated list of secondary members.

### **Primary vs. Secondary Groups**
- **Primary Group**: Set at user creation (in `/etc/passwd`). Files created by the user belong to this group by default.
- **Secondary (Supplementary) Groups**: Additional groups a user can belong to (in `/etc/group`) to gain extra permissions (e.g., `sudo`, `docker`).

### **Management Commands**

#### **Creating Groups (`groupadd`)**
Use `groupadd` to create a new group.
```bash
# Create a group named 'developers'
sudo groupadd developers

# Create a group with a specific GID
sudo groupadd -g 1500 admins
```

#### **Deleting Groups (`groupdel`)**
Use `groupdel` to remove a group.
```bash
# Remove the 'developers' group
sudo groupdel developers
```
> [!IMPORTANT]
> You cannot delete a group that is the primary group of an existing user.

#### **Adding Users to Groups**
To add an existing user to a secondary group, use `usermod`:
```bash
# Add user 'john' to the 'docker' group (append)
sudo usermod -aG docker john
```
*Always use `-a` (append) with `-G` to avoid removing the user from other groups.*

#### **Checking Membership**
```bash
# Show groups for the current user
groups

# Show detailed UID/GID info for a specific user
id username
```

## Interview Questions

**Q: What is the difference between /etc/passwd and /etc/shadow?**
**A:** `/etc/passwd` contains user account metadata and is readable by all users. `/etc/shadow` contains the actual encrypted passwords and sensitive aging information, and is only readable by the root user for security.

**Q: How do you add a user to a group without removing them from their existing groups?**
**A:** You use the `usermod -aG <groupname> <username>` command. The `-a` flag stands for "append," ensuring the user remains in their current supplementary groups.

**Q: What happens to files owned by a group if that group is deleted?**
**A:** The files remain on the system, but their group ownership will show the numeric GID instead of a group name, as the mapping in `/etc/group` no longer exists.

**Q: Can a user have multiple primary groups?**
**A:** No, a user can only have one primary group, which is defined by the GID field in `/etc/passwd`. However, they can belong to multiple secondary (supplementary) groups.

**Q: How can you find which GID corresponds to a group name?**
**A:** You can search the `/etc/group` file using `grep`: `grep "^groupname:" /etc/group`.

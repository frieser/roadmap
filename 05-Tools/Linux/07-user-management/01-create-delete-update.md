#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Linux User Management is a critical aspect of system administration that ensures secure access control. It involves managing user accounts, groups, and permissions using a set of core utilities: \`useradd\` for creation, \`usermod\` for modification, \`userdel\` for deletion, and \`passwd\` for credential management. These tools interact with system configuration files like \`/etc/passwd\` and \`/etc/shadow\` to maintain account integrity.

## Detailed Explanation

### 1. Creating Users (\`useradd\`)
The \`useradd\` command is used to create a new user or update default new user information.

**Basic Syntax:**
\`\`\`bash
sudo useradd [options] username
\`\`\`

**Common Flags:**
- \`-m\`: Creates the user's home directory if it does not exist.
- \`-d /path/to/dir\`: Specifies a custom home directory.
- \`-s /path/to/shell\`: Sets the default login shell (e.g., \`/bin/bash\`).
- \`-g group\`: Sets the user's primary group.
- \`-G group1,group2\`: Adds the user to supplementary groups.

**Example:**
\`\`\`bash
# Create user 'devuser' with a home directory and bash shell
sudo useradd -m -s /bin/bash devuser
\`\`\`

---

### 2. Modifying Users (\`usermod\`)
The \`usermod\` command modifies a user's system account settings.

**Common Operations:**
- **Add to a group:**
  \`\`\`bash
  sudo usermod -aG docker devuser # -a means append, -G for supplementary groups
  \`\`\`
- **Change login name:**
  \`\`\`bash
  sudo usermod -l newname oldname
  \`\`\`
- **Lock/Unlock account:**
  \`\`\`bash
  sudo usermod -L devuser # Locks the account
  sudo usermod -U devuser # Unlocks the account
  \`\`\`
- **Change shell:**
  \`\`\`bash
  sudo usermod -s /bin/zsh devuser
  \`\`\`

---

### 3. Deleting Users (\`userdel\`)
The \`userdel\` command is used to delete a user account and related files.

**Example:**
\`\`\`bash
# Delete user 'devuser' and their home directory
sudo userdel -r devuser
\`\`\`
*Note: Using \`-r\` is recommended to avoid leaving orphaned files owned by the deleted user's UID.*

---

### 4. Password Management (\`passwd\`)
The \`passwd\` command changes passwords for user accounts.

**Examples:**
\`\`\`bash
# Change current user password
passwd

# Change another user's password (as root)
sudo passwd devuser

# Force user to change password at next login
sudo passwd -e devuser

# Check password status
sudo passwd -S devuser
\`\`\`

---

### 5. Key Configuration Files
User data is stored in several plain-text files:
- \`/etc/passwd\`: Contains user account information (UID, GID, home, shell).
- \`/etc/shadow\`: Stores encrypted password information and account expiration details.
- \`/etc/group\`: Contains group definitions and membership.

---

## Interview Questions

**Q: What is the difference between \`useradd\` and \`adduser\`?**
**A:** \`useradd\` is a low-level native binary available on almost all Linux systems. \`adduser\` is often a high-level Perl script (common in Debian/Ubuntu) that acts as a friendly wrapper around \`useradd\`, creating home directories and setting defaults interactively.

**Q: How do you add an existing user to a new group without removing them from their current groups?**
**A:** Use \`usermod\` with the \`-a\` (append) and \`-G\` (groups) flags: \`sudo usermod -aG groupname username\`.

**Q: How can you check if a user account is locked?**
**A:** You can check the \`/etc/shadow\` file; a locked account's password field usually starts with \`!\` or \`*\`. Alternatively, use \`passwd -S username\`.

**Q: What happens if you delete a user without the \`-r\` flag?**
**A:** The user's home directory and files remain on the disk. They will be owned by the numerical UID of the deleted user. If a new user is created later with the same UID, they will automatically gain ownership of those files.

---
---

## Summary
Ansible provides powerful modules for managing the state of files (`copy`, `file`) and software packages (`apt`, `yum`) across an infrastructure. These modules are **idempotent**, meaning they verify state before acting (e.g., only install if not installed).

## Detailed Explanation

### File Management
*   **`copy`**: Uploads a file from Control Node to Managed Node.
    *   `ansible all -m copy -a "src=./app.conf dest=/etc/app.conf mode=0644"`
*   **`file`**: Manages file properties (permissions, ownership, creating directories/symlinks).
    *   `ansible all -m file -a "path=/var/www state=directory"`
*   **`fetch`**: Downloads a file from Managed Node to Control Node (Reverse of copy).

### Package Management
*   **`apt`** (Debian/Ubuntu) / **`yum`** (RHEL/CentOS): Installs/Removes packages.
    *   `ansible all -m apt -a "name=nginx state=present"`
*   **`service`**: Manages service state (start/stop/restart).
    *   `ansible all -m service -a "name=nginx state=started enabled=yes"`

## Go-Specific Context/Examples

A common pattern for Go developers is to compile a binary locally and then use Ansible to deploy it.

### Example: Deploying a Go Binary
```bash
# 1. Build locally
go build -o myapp main.go

# 2. Deploy via Ansible Ad-Hoc
# Copy binary
ansible webservers -m copy -a "src=./myapp dest=/usr/local/bin/myapp mode=0755"

# 3. Ensure service is running (assuming systemd unit exists)
ansible webservers -m service -a "name=myapp state=restarted"
```

## Interview Questions

**Q: What does `state=present` vs `state=latest` mean in package modules?**
**A:**
*   `present`: Installs the package if it's missing. If it's already installed (even an old version), it does nothing. (Safe, stable).
*   `latest`: Checks for updates and upgrades the package to the newest version available in the repo. (Riskier, can break dependencies).

**Q: How do you recursively change permissions on a directory?**
**A:** Use the `file` module with `recurse=yes`.
`ansible all -m file -a "path=/var/www owner=www-data mode=0755 recurse=yes"`

**Q: Can `copy` module handle variable substitution?**
**A:** No, `copy` transfers files exactly as they are. If you need to inject variables (like `{{ db_port }}`) into a file, use the **`template`** module instead (which uses Jinja2).

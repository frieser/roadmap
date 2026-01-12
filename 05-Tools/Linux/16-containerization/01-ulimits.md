---
tags: ['linux', 'roadmap', 'tools']
---

# Ulimits

## Summary
**Ulimits** (User Limits) are a Linux kernel feature used to restrict the resource consumption of processes started by a shell and its child processes. They are essential for system stability, preventing any single user or process from exhausting system resources like memory, CPU time, or file descriptors. These limits can be configured per-session using the `ulimit` command or permanently via `/etc/security/limits.conf`.

## Detailed Explanation

### **Soft vs. Hard Limits**
Linux distinguishes between two types of limits:
- **Soft Limit**: The actual resource limit enforced by the kernel. A user can increase their soft limit up to the value of the hard limit.
- **Hard Limit**: The maximum value a soft limit can reach. Only the `root` user can increase hard limits. Regular users can decrease their hard limits but cannot increase them again once lowered.

### **Common Resource Types**
- **`nofile` (Open Files)**: The maximum number of open file descriptors. Modern applications like databases (PostgreSQL/MySQL) and web servers (Nginx) often require higher values than the default (usually 1024).
- **`nproc` (Max Processes)**: The maximum number of processes or threads a user can create. This helps prevent "fork bombs."
- **`stack`**: The maximum size of the process stack.
- **`memlock`**: The maximum amount of memory that can be locked into RAM to prevent swapping.

### **The `ulimit` Command**
The `ulimit` command is a shell built-in used to view and modify resource limits for the current session.

```bash
# View all current limits (soft by default)
ulimit -a

# View hard limits for all resources
ulimit -Ha

# View the maximum number of open files (soft limit)
ulimit -n

# Set the maximum number of open files to 4096
# Note: This cannot exceed the current hard limit
ulimit -n 4096

# Set the max number of processes for the current user
ulimit -u 1000
```

### **Permanent Configuration**
To make limits persistent across reboots and sessions, modify `/etc/security/limits.conf` or add files to `/etc/security/limits.d/`.

**Format**: `<domain> <type> <item> <value>`

```text
# Examples in /etc/security/limits.conf
*               soft    nofile          4096
*               hard    nofile          8192
@db_admins      soft    nproc           2048
@db_admins      hard    nproc           4096
```

### **Containerization Context**
In containerized environments (Docker/Kubernetes), `ulimits` are crucial because containers share the host's kernel.
- **Docker**: You can set limits using the `--ulimit` flag:
  `docker run --ulimit nofile=1024:2048 alpine`
- **Kubernetes**: Limits are often managed at the node level or via `PodSecurityContext`, though `ulimits` specifically are typically inherited from the container runtime configuration on the host.

## Interview Questions

1. **What is the difference between a soft and a hard limit?**
   - A soft limit is the currently enforced resource limit that a user can increase up to the hard limit. A hard limit is the absolute ceiling set by the system administrator (root) that a regular user cannot exceed or increase.

2. **How do you check all resource limits for the current user session?**
   - You use the `ulimit -a` command. To see hard limits specifically, use `ulimit -Ha`.

3. **Which file is primarily used to set permanent ulimits in Linux?**
   - The `/etc/security/limits.conf` file, which is processed by the `pam_limits` module during user login.

4. **What happens if a process exceeds the `nofile` limit?**
   - The system call to open a new file (like `open()` or `socket()`) will fail, and the application will typically receive a "Too many open files" error (errno EMFILE).

5. **Can a non-root user increase their hard limit?**
   - No. A non-root user can only decrease their hard limit. Only the `root` user (or a user with `CAP_SYS_RESOURCE` capability) can increase a hard limit.

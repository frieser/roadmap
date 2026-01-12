---
tags: ['docker', 'containers', 'security', 'devops', 'tools', 'roadmap']
---

# Runtime Security

## Summary

Docker runtime security involves configuring containers with minimal privileges, implementing defense-in-depth strategies, and following the principle of least privilege. While containers share the host kernel, proper security hardening reduces the attack surface and limits potential damage from container compromises. Key practices include running as non-root, dropping capabilities, using seccomp profiles, AppArmor/SELinux, and resource limits.

## Detailed Explanation

### Linux Capabilities

```bash
# CAPABILITIES - Fine-grained root privileges
# Linux divides root privileges into capabilities
# Containers run with limited capabilities by default

# LIST ALL CAPABILITIES
docker run --rm \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  alpine \
  sh -c 'capsh --print'

# DROP ALL, ADD SPECIFIC
docker run -d \
  --cap-drop=ALL \
  --cap-add=CHOWN \
  --cap-add=DAC_OVERRIDE \
  --cap-add=FOWNER \
  --cap-add=FSETID \
  myapp:latest

# COMMON CAPABILITIES TO DROP
dangerous_caps_to_drop:
  - CAP_SYS_ADMIN      # System administration
  - CAP_SYS_BOOT      # Boot configuration
  - CAP_SYS_CHROOT     # Change root
  - CAP_SYS_MODULE     # Load kernel modules
  - CAP_SYS_PACCT      # Process accounting
  - CAP_SYS_TIME      # Time settings
  - CAP_AUDIT_CONTROL  # Audit logging
  - CAP_MAC_OVERRIDE   # MAC address changes
  - CAP_MKNOD        # Load kernel modules
  - CAP_NET_ADMIN     # Network configuration
  - CAP_NET_RAW       # Raw packet sockets
  - CAP_SYS_PTRACE   # Trace any process
  - CAP_SETUID        # UID/GID manipulation
  - CAP_SETGID        # GID manipulation
  - CAP_SYS_RAWIO     # Raw I/O access

# CAPABILITIES TO KEEP (WEB SERVER)
necessary_web_caps:
  - CAP_NET_BIND_SERVICE  # Bind to ports < 1024
  - CAP_CHOWN           # Change file ownership
  - CAP_DAC_OVERRIDE   # File access control
  - CAP_FSETID         # Set file IDs
  - CAP_FOWNER         # Set file owner
```

### Seccomp Profiles

```bash
# SECCOMP - Secure Computing Mode
# Filters system calls, limits dangerous operations

# VIEW SECCOMP PROFILE
docker run --security-opt seccomp=/path/to/profile.json nginx

# BUILT-IN PROFILES
docker run --security-opt seccomp=docker-default nginx
# Docker's default profile (good baseline)

# STRICT PROFILES
docker run --security-opt seccomp=seccomp-strict nginx
# Very restrictive, may break some apps

# CUSTOM PROFILE
# Create profile JSON:
{
  "defaultAction": "SCMP_ACT_ERR",
  "syscalls": [
    {
      "name": "open",
      "action": "SCMP_ACT_ALLOW"
    },
    {
      "name": "execve",
      "action": "SCMP_ACT_ALLOW"
    },
    {
      "name": "chmod",
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}

# Use custom profile
docker run --security-opt seccomp=profile.json nginx
```

### AppArmor

```bash
# APPARMOR - Mandatory Access Control on Ubuntu
# Linux security module that restricts programs' capabilities

# CHECK APPARMOR STATUS
aa-status
# Shows if AppArmor is running and profiles

# LIST AVAILABLE PROFILES
sudo apparmor_status
sudo apparmor_status | grep docker-default

# VIEW PROFILE CONTENT
sudo apparmor_status docker-default
# Shows the seccomp filter and permissions

# USE APPARMOR PROFILE IN DOCKER
docker run --security-opt apparmor=docker-default nginx
# Or custom profile
docker run --security-opt apparmor=myapp-profile nginx

# CREATE CUSTOM APPARMOR PROFILE
# /etc/apparmor.d/myapp-profile:
#include <tunables/global.d>
#include <sys/klog.h>

profile myapp-profile flags=(attach_disconnected,mediate) {
  # Allow network access
  network inet stream,
  unix stream,

  # Allow basic file operations
  file /etc/** rwm,

  # Deny dangerous operations
  deny /sys/module/** rw,
  deny /proc/kcore/** rw,

  # Allow specific syscalls
  capability dac_override,

}

# Load profile
sudo apparmor_parser -r /etc/apparmor.d/myapp-profile
sudo aa-enforce /etc/apparmor.d/myapp-profile
```

### SELinux

```bash
# SELINUX - Security Enhanced Linux (Red Hat/CentOS)
# Mandatory access control, different from AppArmor

# CHECK SELINUX STATUS
getenforce
# Returns: Enforcing, Permissive, Disabled

# CHECK SELINUX CONTEXT
ls -Z /var/lib/docker/volumes/mydata
# Shows SELinux context label

# CHECK CONTAINER CONTEXT
docker inspect myapp | grep SELinuxContext
# Shows what SELinux context container runs in

# USE SELINUX WITH DOCKER
docker run --security-opt label=user:container_u user_u nginx

# COMMON SELINUX CONTEXTS FOR DOCKER
# container_u - Unconfined containers
# svirt_lxc_net_u - Virtualization containers with network
# svirt_sandbox_file_u - Sandbox containers with file access
# container_file_t - File access inside containers
# spc_t - Shared persistent containers
```

### User Namespaces

```bash
# USER NAMESPACES - Map container users to host users
# Root in container ≠ Root on host
# Provides security and proper file permissions

# ENABLE USER NAMESPACES
# /etc/subuid:
root:100000:65536
# Maps root UID 0 to host UID 100000
# Users 100000-65535 run as root inside container

# Use user namespace
docker run -d \
  --name myapp \
  -v $(pwd):/app \
  alpine
# root@100000 inside container = current user on host

# VERIFY MAPPING
docker exec myapp id
# uid=0(root) gid=0(root)  # Inside container
```

### Rootless Mode

```bash
# ROOTLESS DOCKER - Runs without root privileges
# Better security, no Docker daemon root required

# SETUP ROOTLESS DOCKER
dockerr-rootless-setuptool.sh install

# Start rootless daemon
systemctl --user start docker

# Configure environment
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

# Use rootless Docker normally
docker run nginx
docker build -t myapp:latest .
# Works with user's permissions, no sudo needed

# LIMITATIONS OF ROOTLESS MODE
- Cannot use --privileged mode
- Cannot mount /var/run/docker.sock (for Docker-in-Docker)
- Cannot bind ports < 1024 (unless configured)
- No host networking (--network=host)
- Limited cgroup configuration
```

### Read-Only Root Filesystem

```bash
# READ-ONLY ROOT FS
docker run -d \
  --read-only \
  --tmpfs /tmp \
  myapp:latest

# READ-ONLY WITH WRITABLE /TMP
docker run -d \
  --read-only \
  -v /tmp:/tmp \
  myapp:latest

# BENEFITS
- Prevents malicious code injection
- Stops accidental modifications
- Forces applications to write to specific directories (volumes, /tmp)

# CONSIDERATIONS
- /tmp must be used for writable data
- Logs must go to volume or stdout
- State must be external (databases, object storage)
```

### No-New-Privileges

```bash
# NO NEW PRIVILEGES
docker run -d \
  --security-opt no-new-privileges=true \
  myapp:latest

# WHAT IT DISABLES:
- Cannot gain new capabilities (even from parent)
- Cannot change UID/GID
- Cannot change seccomp profile to allow more syscalls
- Cannot execute SUID/GUID binaries
- Disallows certain kernel module loading

# USE IN PRODUCTION
# Recommended for high-security workloads
# Reduces attack surface
# Requires proper security design
```

### Resource Limits

```bash
# MEMORY LIMITS
docker run -d \
  --memory=512m \
  --memory-swap=512m \
  myapp:latest

# CPU LIMITS
docker run -d \
  --cpus=1.0 \
  --cpu-shares=512 \
  myapp:latest

# PID LIMITS (PREVENT FORK BOMBS)
docker run -d \
  --pids-limit=100 \
  myapp:latest

# Combined limits for production
docker run -d \
  --memory=1g \
  --cpus=1.0 \
  --pids-limit=500 \
  --restart unless-stopped \
  myapp:latest
```

### Network Isolation

```bash
# USER-DEFINED BRIDGE NETWORKS
docker network create --driver bridge mysecure-net

# RUN IN ISOLATED NETWORK
docker run -d \
  --network mysecure-net \
  --name isolated-app \
  myapp:latest

# DISABLE NETWORK EXTERNAL ACCESS
docker run -d \
  --network=none \
  myapp:latest
# Container has no network access

# MACVLAN / IPVLAN FOR ADVANCED ISOLATION
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --opt parent=eth0 \
  mymacvlan

docker run -d \
  --network mymacvlan \
  myapp:latest
```

### Docker Desktop Security

```bash
# DOCKER DESKTOP SETTINGS (Mac/Windows)

# Enable Vulnerability Scanning
Settings → Docker Engine → Enable Docker Scout

# Scan images before running
# Right-click image → Scan image

# View scan results
# docker scout cves myapp:latest

# Configure automatic scanning
# Settings → Docker Engine → Scan images on pull

# FILE SHARING SECURITY
# Settings → Resources → File sharing
# Limit to specific directories
# Use Read-Only where possible

# CREDENTIALS HELPER
# Don't save credentials in keychain
# Use Docker context per project

# UPDATE DOCKER DESKTOP REGULARLY
# Security patches released frequently
```

### Vulnerability Scanning

```bash
# DOCKER SCOUT (BUILT-IN)
docker scout cves myapp:latest
docker scout quickview myapp:latest
docker scout recommendations myapp:latest

# Scan before deployment
docker scout cves myapp:latest
# Block deployment if critical CVEs found

# Trivy (THIRD PARTY)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest \
  image myapp:latest

# Grype (ANOTHER OPTION)
docker run --rm \
  anchore/grype:latest \
  myapp:latest

# Integrate into CI/CD
# GitHub Actions: Use trivy-action
# GitLab CI: Use container scanning job
```

### Secrets Management

```bash
# NEVER STORE SECRETS IN IMAGES
# BAD:
# ARG DATABASE_URL="postgres://user:pass@db"
# ENV API_KEY="secret123"

# GOOD: USE RUNTIME SECRETS
docker run -e DATABASE_URL=$DATABASE_URL myapp
docker run --env-file .env myapp

# DOCKER SECRETS (BUILD-TIME)
docker build \
  --secret id=mysecret,src=secrets.txt \
  -t myapp:latest .
# In Dockerfile:
# RUN --mount=type=secret,id=mysecret,target=/app/secret,uid=1000 \
  echo "Setting up secret..."

# KUBERNETES SECRETS
kubectl create secret docker-registry myregistry-config
# In pod spec:
# envFrom:
#   - secretRef:
#       name: docker-registry-secret
```

## Interview Questions

### Q1: What is the principle of least privilege?
**A:** Only give containers the minimum capabilities needed for their function. Drop all capabilities, then add back only specific ones (like `CAP_NET_BIND_SERVICE` for web servers). This reduces attack surface and limits potential damage from compromise.

### Q2: What is the difference between AppArmor and SELinux?
**A:** Both are mandatory access control systems but different approaches. AppArmor uses profiles with file paths (Ubuntu, Debian, SUSE). SELinux uses contexts with labels (Red Hat, CentOS). Both can restrict containers, but only one runs on a given system at a time.

### Q3: What is rootless Docker and why is it more secure?
**A:** Rootless Docker runs containers without requiring root privileges on the host. It uses user namespaces to map root UID inside the container to an unprivileged user on the host. This provides better security isolation and doesn't require sudo or a running Docker daemon as root.

### Q4: What does `--read-only` flag do?
**A:** It makes the container's root filesystem read-only, preventing modifications to files and directories. This is a strong security measure but requires writable directories for logs, temporary files, and application data (which should use volumes).

### Q5: What is seccomp and how does it improve container security?
**A:** Seccomp (Secure Computing Mode) is a Linux kernel feature that filters system calls. It can block dangerous operations (like `chmod`, `chown`, `mount`) even if the container has those capabilities. Docker provides default profiles (`docker-default`, `seccomp-strict`) and supports custom profiles.

### Q6: What is the purpose of user namespaces?
**A:** User namespaces allow mapping user IDs between container and host. Root inside the container (UID 0) can be mapped to a non-root user on the host. This prevents privilege escalation if a container is compromised - the attacker only gains the host user's privileges, not root.

### Q7: How do you properly manage secrets in Docker?
**A:** Never hardcode secrets in Dockerfiles or build arguments. Use environment variables at runtime (`-e SECRET=$VALUE`), Docker secrets (`--secret` build arg), or Kubernetes secrets (which mount as files). Use `.dockerignore` to prevent accidental secret inclusion.

### Q8: What is the difference between `--cap-drop=ALL` and no-new-privileges?
**A:** `--cap-drop=ALL` removes all Linux capabilities from the container, which is a good security baseline. `--no-new-privileges` goes further by preventing the container from gaining ANY new capabilities, even from parent processes. Use both for maximum security.

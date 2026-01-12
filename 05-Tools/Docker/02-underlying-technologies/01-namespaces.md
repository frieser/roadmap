---
tags: ['docker', 'containers', 'linux', 'devops', 'tools', 'roadmap']
---

# Linux Namespaces

## Summary

Linux namespaces are a kernel feature that provides process isolation by creating separate instances of global system resources. Each namespace type isolates a specific aspect: process IDs, network stack, mount points, users, hostnames, and inter-process communication. Containers use namespaces to create the illusion that each container has its own isolated system, even though they share the same kernel.

## Detailed Explanation

### What Namespaces Provide

```
HOST SYSTEM VIEW:
┌─────────────────────────────────────────────────────────────┐
│ PID 1: systemd                                              │
│ PID 100: dockerd                                            │
│ PID 200: containerd                                         │
│ PID 300: nginx (container A)                                │
│ PID 301: nginx worker (container A)                         │
│ PID 400: redis (container B)                                │
└─────────────────────────────────────────────────────────────┘

CONTAINER A VIEW (pid namespace):
┌─────────────────────────────────────────────────────────────┐
│ PID 1: nginx                                                │
│ PID 2: nginx worker                                         │
│ (Cannot see host processes or container B)                  │
└─────────────────────────────────────────────────────────────┘

CONTAINER B VIEW (pid namespace):
┌─────────────────────────────────────────────────────────────┐
│ PID 1: redis                                                │
│ (Cannot see host processes or container A)                  │
└─────────────────────────────────────────────────────────────┘
```

### Types of Namespaces

```bash
# Linux provides 8 namespace types:

# 1. PID (Process ID) - CLONE_NEWPID
#    Isolates process ID number space
#    Container processes see themselves starting from PID 1

# 2. NET (Network) - CLONE_NEWNET
#    Isolates network stack (interfaces, routes, iptables)
#    Container has own network interfaces

# 3. MNT (Mount) - CLONE_NEWNS
#    Isolates mount points
#    Container sees only its own filesystem

# 4. UTS (Unix Time Sharing) - CLONE_NEWUTS
#    Isolates hostname and domain name
#    Container can have its own hostname

# 5. IPC (Inter-Process Communication) - CLONE_NEWIPC
#    Isolates System V IPC and POSIX message queues
#    Container IPC resources are isolated

# 6. USER - CLONE_NEWUSER
#    Isolates user and group IDs
#    Root in container can be non-root on host

# 7. CGROUP - CLONE_NEWCGROUP
#    Isolates cgroup root directory
#    Container sees isolated cgroup view

# 8. TIME - CLONE_NEWTIME (Linux 5.6+)
#    Isolates system clocks
#    Container can have different time offsets

# View namespaces for a process
ls -la /proc/$$/ns/
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 cgroup -> 'cgroup:[4026531835]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 ipc -> 'ipc:[4026531839]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 mnt -> 'mnt:[4026531840]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 net -> 'net:[4026531993]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 pid -> 'pid:[4026531836]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 user -> 'user:[4026531837]'
# lrwxrwxrwx 1 user user 0 Jan  1 00:00 uts -> 'uts:[4026531838]'
```

### PID Namespace

```bash
# PID namespace isolates process ID numbers
# First process in namespace becomes PID 1

# Create new PID namespace
sudo unshare --pid --fork --mount-proc bash

# Inside new namespace
ps aux
# USER  PID %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
# root    1  0.0  0.0  18504  3352 pts/0 S    00:00   0:00 bash
# root    2  0.0  0.0  34400  2848 pts/0 R+   00:00   0:00 ps aux

# Container process becomes PID 1
# PID 1 has special signal handling (init process)

# Docker uses this:
docker run --rm alpine ps aux
# PID   USER     TIME  COMMAND
#     1 root      0:00 ps aux

# Compare with host
docker inspect $(docker run -d alpine sleep 1000) | grep -i pid
# "Pid": 12345
# Container sees PID 1, host sees PID 12345
```

### Network Namespace

```bash
# Network namespace provides isolated network stack

# Create network namespace
sudo ip netns add myns

# List namespaces
ip netns list
# myns

# Execute in namespace
sudo ip netns exec myns ip link
# 1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN
# (Only loopback, no other interfaces)

# Create veth pair to connect namespaces
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns myns

# Configure interfaces
sudo ip addr add 10.0.0.1/24 dev veth0
sudo ip link set veth0 up
sudo ip netns exec myns ip addr add 10.0.0.2/24 dev veth1
sudo ip netns exec myns ip link set veth1 up
sudo ip netns exec myns ip link set lo up

# Test connectivity
sudo ip netns exec myns ping 10.0.0.1

# Docker creates network namespaces automatically
docker run --rm alpine ip addr
# Shows container's network interfaces

# Cleanup
sudo ip netns delete myns
```

### Mount Namespace

```bash
# Mount namespace isolates mount points
# Container has its own filesystem view

# Create mount namespace
sudo unshare --mount bash

# Mounts here don't affect host
mount -t tmpfs tmpfs /mnt
# This mount is only visible in this namespace

# Docker uses mount namespace + overlay filesystem
# Container sees:
# /  - Container's root filesystem
# /etc, /bin, etc. - From image layers
# /dev - Device files
# /proc - Process filesystem
# /sys - Sysfs

# Docker mount options
docker run -v /host/path:/container/path alpine ls /container/path
# Bind mount from host to container

docker run --mount type=tmpfs,destination=/app/tmp alpine df -h
# tmpfs mount in container
```

### User Namespace

```bash
# User namespace maps user IDs
# Root inside container can be non-root outside

# Enable user namespaces (rootless containers)
# Container UID 0 → Host UID 100000
# Container UID 1 → Host UID 100001

# Create user namespace
unshare --user --map-root-user bash

# Inside namespace
id
# uid=0(root) gid=0(root) groups=0(root)

# But on host, it's running as regular user

# Docker rootless mode uses user namespaces
dockerd-rootless-setuptool.sh install

# Podman uses user namespaces by default
podman run --rm alpine id
# uid=0(root) gid=0(root) <- inside container
# But not actual root on host

# Benefits:
# - Security: Container root can't escalate to host root
# - Allows unprivileged users to run containers
# - Limits blast radius of container escape
```

### UTS and IPC Namespaces

```bash
# UTS Namespace - hostname isolation
sudo unshare --uts bash
hostname mycontainer
hostname
# mycontainer (doesn't affect host)

# Docker sets container hostname
docker run --rm alpine hostname
# Random container ID

docker run --rm --hostname myapp alpine hostname
# myapp

# IPC Namespace - isolate inter-process communication
# System V IPC: shared memory, semaphores, message queues
# POSIX message queues

# Each container has isolated IPC
docker run --rm alpine ipcs
# Shows only this container's IPC resources

# Share IPC namespace between containers
docker run -d --name=producer --ipc=shareable alpine sleep 1000
docker run --rm --ipc=container:producer alpine ipcs
# Shares IPC namespace with producer container
```

### Namespaces in Docker

```bash
# Docker creates namespaces for each container
# Default: all namespaces except user (configurable)

# View container namespaces
docker run -d --name test alpine sleep 1000
docker inspect test --format '{{.State.Pid}}'
# 12345

sudo ls -la /proc/12345/ns/
# Shows container's namespace inodes

# Sharing namespaces
# Share network namespace (same IP)
docker run -d --name web nginx
docker run --rm --net=container:web alpine wget -qO- localhost

# Share PID namespace (see other container's processes)
docker run -d --name app alpine sleep 1000
docker run --rm --pid=container:app alpine ps aux
# Sees app container's processes

# Host namespace (no isolation)
docker run --rm --net=host alpine ip addr
# Shows host's network interfaces

docker run --rm --pid=host alpine ps aux
# Shows all host processes (dangerous!)

# Privileged mode (all namespaces + capabilities)
docker run --rm --privileged alpine ls /dev
# Full device access (very dangerous!)
```

### Creating Namespaces Programmatically

```c
// C code to create namespaces (simplified)
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h>

int main() {
    // Create new namespaces
    unshare(CLONE_NEWPID | CLONE_NEWNET | CLONE_NEWNS | 
            CLONE_NEWUTS | CLONE_NEWIPC);
    
    // Fork to enter new PID namespace
    if (fork() == 0) {
        // Child is PID 1 in new namespace
        sethostname("container", 9);
        execl("/bin/bash", "bash", NULL);
    }
    return 0;
}
```

```bash
# Go code is used by Docker/containerd
# Uses syscalls: clone(), unshare(), setns()

# Enter existing namespace
sudo nsenter --target $PID --mount --uts --ipc --net --pid bash
# Enters all namespaces of process $PID
```

## Interview Questions

### Q1: What are Linux namespaces?
**A:** Namespaces are a kernel feature that isolates global system resources into separate instances. Each namespace type isolates a different resource (PIDs, network, mounts, users, etc.), creating the illusion of isolated systems while sharing the same kernel.

### Q2: Name the main namespace types used by containers.
**A:** PID (process IDs), NET (network stack), MNT (mount points), UTS (hostname), IPC (inter-process communication), USER (user/group IDs), CGROUP (cgroup root), and TIME (system clocks - newer).

### Q3: Why is PID 1 special in containers?
**A:** PID 1 is the init process with special signal handling - it won't be killed by signals that lack explicit handlers. Container entrypoints become PID 1 and should properly handle signals and reap zombie processes.

### Q4: What is the user namespace used for?
**A:** User namespace maps container UIDs/GIDs to different host IDs. Root inside the container (UID 0) can map to an unprivileged user on the host, enabling rootless containers and limiting damage from container escapes.

### Q5: How does network namespace isolation work?
**A:** Each network namespace has its own network interfaces, IP addresses, routing tables, and iptables rules. Docker creates veth pairs connecting container namespaces to a bridge network for communication.

### Q6: What does `--net=host` do?
**A:** It runs the container in the host's network namespace with no network isolation. The container shares the host's IP addresses and ports. Useful for performance but removes network security boundaries.

### Q7: How can you enter a container's namespaces?
**A:** Use `nsenter --target $PID --all` or `docker exec`. These use the `setns()` syscall to join existing namespaces. Useful for debugging containers.

### Q8: Do namespaces provide complete isolation?
**A:** No. All containers share the same kernel, so kernel vulnerabilities can affect all containers. For stronger isolation, combine namespaces with seccomp, capabilities, SELinux/AppArmor, or use VM-based runtimes like gVisor or Kata.

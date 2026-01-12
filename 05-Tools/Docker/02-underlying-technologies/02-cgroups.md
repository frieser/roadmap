---
tags: ['docker', 'containers', 'linux', 'devops', 'tools', 'roadmap']
---

# Control Groups (cgroups)

## Summary

Control Groups (cgroups) are a Linux kernel feature that limits, accounts for, and isolates resource usage (CPU, memory, disk I/O, network) of process groups. While namespaces provide isolation (what a process can see), cgroups provide resource limits (how much a process can use). Docker uses cgroups to prevent containers from consuming excessive resources and affecting other containers or the host system.

## Detailed Explanation

### What cgroups Control

```
┌─────────────────────────────────────────────────────────────┐
│                      HOST RESOURCES                          │
│  CPU: 8 cores    Memory: 32GB    Disk I/O    Network I/O    │
└─────────────────────────────────────────────────────────────┘
                            │
                    cgroups allocation
                            │
     ┌──────────────────────┼──────────────────────┐
     │                      │                      │
     ▼                      ▼                      ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ Container A │      │ Container B │      │ Container C │
│ CPU: 2 cores│      │ CPU: 4 cores│      │ CPU: 2 cores│
│ Mem: 4GB    │      │ Mem: 16GB   │      │ Mem: 8GB    │
│ I/O: limited│      │ I/O: limited│      │ I/O: limited│
└─────────────┘      └─────────────┘      └─────────────┘
```

### cgroup Versions

```bash
# cgroups v1 (legacy)
# - Multiple hierarchies
# - Each controller independent
# - Used by older systems

# cgroups v2 (unified)
# - Single unified hierarchy
# - Better resource control
# - Default on modern systems

# Check which version is in use
mount | grep cgroup
# cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime)
# ^ This indicates cgroups v2

# Or check for cgroups v1
ls /sys/fs/cgroup/
# If you see: cpu, memory, blkio, etc. → v1
# If you see: cgroup.controllers, cgroup.procs → v2

# Docker cgroups driver
docker info | grep -i cgroup
# Cgroup Driver: systemd
# Cgroup Version: 2
```

### cgroup Controllers

```bash
# Main controllers for containers:

# 1. CPU - Limit CPU usage
# cpu.max - Maximum CPU time (v2)
# cpu.shares - Relative CPU weight (v1/v2)

# 2. MEMORY - Limit memory usage  
# memory.max - Hard memory limit
# memory.high - Memory pressure threshold
# memory.swap.max - Swap limit

# 3. IO (blkio in v1) - Limit disk I/O
# io.max - IOPS and bandwidth limits
# io.weight - Relative I/O priority

# 4. PIDS - Limit number of processes
# pids.max - Maximum number of PIDs

# 5. CPUSET - Pin to specific CPUs
# cpuset.cpus - Which CPUs can be used
# cpuset.mems - Which memory nodes

# View available controllers
cat /sys/fs/cgroup/cgroup.controllers
# cpuset cpu io memory hugetlb pids rdma
```

### Memory Limits

```bash
# Docker memory limits
docker run -d --name limited \
  --memory=512m \
  --memory-swap=1g \
  --memory-reservation=256m \
  alpine sleep 1000

# Breakdown:
# --memory=512m       Hard limit (OOM killer triggers)
# --memory-swap=1g    Memory + swap total
# --memory-reservation=256m  Soft limit (hint for scheduler)

# View container memory usage
docker stats limited
# CONTAINER   MEM USAGE / LIMIT    MEM %
# limited     1.5MiB / 512MiB      0.29%

# What happens at limit?
# Container process killed by OOM (Out of Memory) killer

# View cgroup settings
docker inspect limited --format '{{.HostConfig.Memory}}'
# 536870912 (bytes = 512MB)

# Actual cgroup file (v2)
cat /sys/fs/cgroup/docker/<container_id>/memory.max
# 536870912
```

```bash
# Manual cgroup memory limit (educational)
# Create cgroup
sudo mkdir /sys/fs/cgroup/mygroup

# Set memory limit to 100MB
echo 104857600 | sudo tee /sys/fs/cgroup/mygroup/memory.max

# Add process to cgroup
echo $$ | sudo tee /sys/fs/cgroup/mygroup/cgroup.procs

# Process is now limited to 100MB
```

### CPU Limits

```bash
# Docker CPU limits
docker run -d --name cpulimited \
  --cpus=1.5 \
  --cpu-shares=512 \
  --cpuset-cpus=0,1 \
  alpine stress --cpu 4

# Breakdown:
# --cpus=1.5          Limit to 1.5 CPU cores worth of time
# --cpu-shares=512    Relative weight (default 1024)
# --cpuset-cpus=0,1   Pin to CPU cores 0 and 1

# CPU shares explained:
# Container A: --cpu-shares=1024
# Container B: --cpu-shares=512
# Under contention: A gets 2x CPU time of B
# When idle: either can use 100%

# View CPU usage
docker stats cpulimited

# cgroup v2 CPU settings
# cpu.max format: "quota period"
# "150000 100000" means 1.5 CPUs (150ms per 100ms period)
cat /sys/fs/cgroup/docker/<container_id>/cpu.max
# 150000 100000
```

```bash
# CPU period and quota (advanced)
docker run -d \
  --cpu-period=100000 \
  --cpu-quota=150000 \
  alpine stress --cpu 4

# cpu-quota / cpu-period = CPU limit
# 150000 / 100000 = 1.5 CPUs
```

### I/O Limits

```bash
# Docker I/O limits
docker run -d --name iolimited \
  --device-read-bps=/dev/sda:10mb \
  --device-write-bps=/dev/sda:10mb \
  --device-read-iops=/dev/sda:1000 \
  --device-write-iops=/dev/sda:1000 \
  alpine dd if=/dev/zero of=/testfile bs=1M count=100

# Breakdown:
# --device-read-bps   Read bandwidth limit
# --device-write-bps  Write bandwidth limit
# --device-read-iops  Read IOPS limit
# --device-write-iops Write IOPS limit

# I/O weight (relative priority)
docker run -d --blkio-weight=500 alpine

# Note: I/O limits work best with direct I/O
# Buffered I/O may not respect limits immediately
```

### PID Limits

```bash
# Limit number of processes (fork bomb protection)
docker run -d --name pidlimited \
  --pids-limit=100 \
  alpine sleep 1000

# Try to create too many processes
docker exec pidlimited sh -c 'for i in $(seq 1 200); do sleep 1000 & done'
# Resource temporarily unavailable (cannot fork)

# View limit
cat /sys/fs/cgroup/docker/<container_id>/pids.max
# 100

# Docker default: no limit (dangerous!)
# Kubernetes default: per-pod limit
```

### Viewing Container Resource Usage

```bash
# Docker stats (real-time)
docker stats
# CONTAINER   CPU %  MEM USAGE/LIMIT   MEM %  NET I/O      BLOCK I/O
# web         0.50%  50MiB/512MiB      9.77%  1.5kB/0B     0B/0B
# db          2.30%  200MiB/1GiB       19.5%  5kB/2kB      10MB/5MB

# Single container with no stream
docker stats --no-stream web

# Container resource inspection
docker inspect web --format '
  Memory Limit: {{.HostConfig.Memory}}
  CPU Shares: {{.HostConfig.CpuShares}}
  CPU Period: {{.HostConfig.CpuPeriod}}
  CPU Quota: {{.HostConfig.CpuQuota}}
'

# Raw cgroup files
ls /sys/fs/cgroup/docker/<container_id>/
# cgroup.procs  cpu.max  cpu.stat  memory.current  memory.max  ...
```

### cgroups and Kubernetes

```yaml
# Kubernetes resource requests and limits
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"    # Scheduling decision
        cpu: "250m"       # 0.25 CPU cores
      limits:
        memory: "128Mi"   # cgroup memory.max
        cpu: "500m"       # cgroup cpu.max

# requests: Used for scheduling (guaranteed minimum)
# limits: Enforced by cgroups (hard maximum)

# CPU units in Kubernetes:
# 1 = 1 CPU core
# 100m = 0.1 CPU core (millicores)
# 1000m = 1 CPU core

# Memory exceeded → Container killed (OOMKilled)
# CPU exceeded → Container throttled (not killed)
```

### Best Practices

```yaml
# 1. ALWAYS SET LIMITS
# Prevents runaway containers from affecting others
docker run --memory=512m --cpus=1 myapp

# 2. SET REASONABLE DEFAULTS
# Don't over-provision
# Start low, increase based on monitoring

# 3. MONITOR RESOURCE USAGE
# Use docker stats, cAdvisor, Prometheus
# Set alerts for approaching limits

# 4. UNDERSTAND OOM BEHAVIOR
# memory.oom_control - what happens at limit
# By default: OOM killer terminates processes
# Consider: memory.high for soft throttling first

# 5. CONSIDER SWAP SETTINGS
# --memory-swap=0  Disable swap for container
# --memory-swappiness=0  Prefer not to swap
# Generally: disable swap for predictable performance

# 6. USE APPROPRIATE PID LIMITS
# Protect against fork bombs
# docker run --pids-limit=256 myapp
```

## Interview Questions

### Q1: What is the difference between namespaces and cgroups?
**A:** Namespaces provide isolation - what a process can see (own PIDs, network, filesystem). Cgroups provide resource limits - how much a process can use (CPU, memory, I/O). Containers need both for proper isolation.

### Q2: What happens when a container exceeds its memory limit?
**A:** The Linux OOM (Out of Memory) killer terminates processes in the container. In Docker, this typically kills the main container process, causing the container to stop. Kubernetes marks the pod as "OOMKilled".

### Q3: What happens when a container exceeds its CPU limit?
**A:** The container is throttled, not killed. The cgroup scheduler limits the container's CPU time to the specified quota. The container runs slower but continues running.

### Q4: What is the difference between cpu-shares and cpus?
**A:** `--cpu-shares` sets relative weight (matters only under contention). `--cpus` sets an absolute limit (e.g., 1.5 CPU cores maximum). Shares are soft limits; cpus is a hard limit.

### Q5: What is the difference between cgroups v1 and v2?
**A:** cgroups v1 uses multiple hierarchies (one per controller). cgroups v2 uses a single unified hierarchy, provides better resource control, and is the default on modern systems. Docker and Kubernetes support both.

### Q6: How do you protect against fork bombs in containers?
**A:** Use `--pids-limit` to restrict the maximum number of processes a container can create. Without this limit, a malicious container could exhaust the system's PID space.

### Q7: What is the purpose of memory.high vs memory.max?
**A:** `memory.max` is a hard limit - exceeding triggers OOM killer. `memory.high` is a soft limit - exceeding causes throttling and reclaim pressure, giving applications time to reduce usage before OOM.

### Q8: How do Kubernetes requests differ from limits?
**A:** Requests are the guaranteed minimum used for pod scheduling decisions. Limits are hard maximums enforced by cgroups. A pod can use more than its request (up to limit) if resources are available.

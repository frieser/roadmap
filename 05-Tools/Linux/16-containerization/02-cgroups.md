---
tags: ['linux', 'roadmap']
---

# Control Groups (cgroups)

## Summary
Control Groups (cgroups) is a Linux kernel feature that allows organizing processes into hierarchical groups and distributing system resources (such as CPU time, system memory, network bandwidth, or combinations of these) among those groups. It is one of the fundamental pillars of modern containerization technologies like Docker and Kubernetes, providing the mechanism for resource isolation, limitation, and accounting.

## Detailed Explanation

### Core Concepts
cgroups provide several key features for system administration and containerization:
- **Resource Limiting**: Groups can be set to not exceed a certain memory limit or CPU share.
- **Prioritization**: Some groups may get a larger share of resources (CPU, I/O) when the system is under load.
- **Accounting**: Measures how much resource a certain group uses (useful for monitoring and billing).
- **Control**: Freezing (suspending) or restarting groups of processes.

### cgroups v1 vs v2
- **cgroups v1**: Introduced multiple hierarchies, where each controller (CPU, Memory, etc.) lived in its own tree. This led to complexity when trying to coordinate resources across different controllers (e.g., attributing I/O to a specific memory-limited process).
- **cgroups v2**: Introduced a **unified hierarchy**. All controllers are managed under a single tree structure. It simplifies management and fixes several design flaws in v1, such as inconsistent resource accounting for buffered I/O and easier delegation to non-root users.

### Main Controllers
- **cpu**: Controls CPU usage via weights (proportional) and maximum limits (absolute).
- **memory**: Controls memory usage, including hard limits (`memory.max`), soft limits (`memory.high`), and swap control.
- **io**: Controls block I/O, allowing limits on bandwidth (BPS) or operations per second (IOPS).
- **pids**: Limits the number of processes/threads that can be created in a cgroup, effectively preventing fork bombs.
- **cpuset**: Binds cgroups to specific CPUs or memory nodes.

### Bash Implementation (cgroups v2)
In modern Linux distributions, cgroups v2 is the default. It is managed via the `cgroup2` filesystem, typically mounted at `/sys/fs/cgroup`.

#### 1. Checking the cgroup Version
```bash
# If the output includes 'cgroup2', the system is using v2
mount | grep cgroup
```

#### 2. Creating a Hierarchy
To create a cgroup, simply create a directory within the cgroup mount point:
```bash
sudo mkdir /sys/fs/cgroup/workload
```

#### 3. Enabling Controllers
A new cgroup does not automatically enable controllers for its sub-groups. You must explicitly enable them in the `cgroup.subtree_control` file:
```bash
# Enable memory and pids controllers for any children of 'workload'
echo "+memory +pids" | sudo tee /sys/fs/cgroup/workload/cgroup.subtree_control
```

#### 4. Setting Resource Limits
Create a child cgroup and apply limits:
```bash
sudo mkdir /sys/fs/cgroup/workload/app-instance
# Limit memory to 256MB
echo "256M" | sudo tee /sys/fs/cgroup/workload/app-instance/memory.max
# Limit to 100 processes
echo "100" | sudo tee /sys/fs/cgroup/workload/app-instance/pids.max
```

#### 5. Assigning a Process
To move a process into the cgroup, write its PID to the `cgroup.procs` file:
```bash
# Move current shell to the cgroup
echo $$ | sudo tee /sys/fs/cgroup/workload/app-instance/cgroup.procs
```

## Interview Questions

**Q: What is the primary difference between cgroups v1 and cgroups v2?**
**A:** The main difference is the hierarchy model. v1 uses multiple hierarchies (one per resource controller), which makes it difficult to coordinate resources. v2 uses a **unified hierarchy** where all controllers are attached to a single tree. v2 also fixes issues with buffered I/O accounting and provides a cleaner, more consistent interface.

**Q: How does `cgroup.subtree_control` work in v2?**
**A:** It determines which controllers are enabled for the **immediate children** of a cgroup. A controller must be enabled in the parent's `cgroup.subtree_control` (using `+name`) before it can be configured in the child's interface files.

**Q: What is the "No Internal Process" constraint in cgroups v2?**
**A:** This rule states that a non-root cgroup cannot have both member processes and active controllers in its `cgroup.subtree_control`. This prevents the ambiguity of how resources would be shared between a parent's own processes and its child cgroups. To control a process, you should move it to a leaf cgroup.

**Q: How do container runtimes like Docker utilize cgroups?**
**A:** When a container is started with resource constraints (e.g., `docker run --memory=512m`), the runtime creates a dedicated cgroup directory for that container and writes the limits to the corresponding control files (like `memory.max`). This ensures the kernel enforces the limits on all processes within that container.

**Q: How can you verify which cgroup a specific process belongs to?**
**A:** By checking the `/proc/<PID>/cgroup` file. For cgroups v2, it will typically show a line starting with `0::/` followed by the path to the cgroup relative to the mount point.

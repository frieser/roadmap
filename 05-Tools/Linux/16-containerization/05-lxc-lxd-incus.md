---
tags: ['linux', 'roadmap']
---

## Summary
**LXC (Linux Containers)** and its managers **LXD** (and the community fork **Incus**) provide "System Containers" that behave like lightweight Virtual Machines. Unlike Docker, which focuses on single-process "Application Containers," LXC provides a full init system (like systemd) inside the container, allowing for multiple processes, cron jobs, and logging to run exactly as they would on a standalone Linux server but with the efficiency of shared kernel resources.

## Detailed Explanation

### 1. OS-Level Virtualization vs. Application Containers
In the modern container ecosystem, it is critical to distinguish between these two approaches:

*   **Application Containers (Docker/Podman)**: Designed to package and run a single application or process. They are ephemeral, stateless by default, and typically do not include an init system (PID 1 is the app itself).
*   **System Containers (LXC/Incus)**: Designed to run a full Linux Operating System. They include an init system (systemd, OpenRC), multiple services, and persistent configuration. They are often used as "VM-lite" to replace full Virtual Machines when kernel-level isolation is sufficient.

### 2. Architecture: Kernel Primitives
LXC is essentially a userspace interface for the Linux kernel's containment features. It relies on three primary pillars:

*   **Namespaces**: Provide **isolation**. They ensure a process in a container cannot see processes, network interfaces, or mount points outside its own namespace.
*   **Cgroups (Control Groups)**: Provide **resource limiting**. They control how much CPU, memory, and I/O a container can consume.
*   **Security Modules (AppArmor/Seccomp)**: Provide **privilege restriction**. They limit the syscalls a container can make to the host kernel, preventing breakouts.

### 3. LXC, LXD, and the Incus Fork
*   **LXC**: The low-level project providing tools and libraries to create containers.
*   **LXD**: Originally created by Canonical. It provides a REST API, image management, and clustering for LXC. In 2023, Canonical moved LXD to the AGPLv3 license and took it out of the Linux Containers project.
*   **Incus**: A community-driven fork of LXD created by the original LXD maintainers. It remains under the Apache 2.0 license and is the preferred choice for open-source enthusiasts and those avoiding Canonical's licensing changes.

### 4. Practical Usage (Bash Examples)

If you are using **Incus** (the modern community standard), the CLI is `incus`. If using LXD, replace `incus` with `lxc`.

#### Basic Lifecycle
```bash
# Launch a new Ubuntu 24.04 system container
incus launch images:ubuntu/24.04 my-server

# List running containers
incus list

# Execute a shell inside the container
incus exec my-server -- bash

# Stop and delete the container
incus stop my-server
incus delete my-server
```

#### Resource Management
```bash
# Limit a container to 2 CPUs and 1GB of RAM
incus config set my-server limits.cpu 2
incus config set my-server limits.memory 1GiB
```

### 5. Go Integration
Since LXD and Incus are written in **Go**, they provide excellent client libraries for programmatic management.

```go
package main

import (
	"fmt"
	"github.com/lxc/incus/client"
	"github.com/lxc/incus/shared/api"
)

func main() {
	// Connect to the local Incus daemon via Unix socket
	c, err := incus.ConnectIncusUnix("", nil)
	if err != nil {
		panic(err)
	}

	// Define a new system container
	req := api.InstancesPost{
		Name: "dev-server",
		Source: api.InstanceSource{
			Type:  "image",
			Alias: "ubuntu/24.04",
		},
	}

	// Create the container
	op, err := c.CreateInstance(req)
	if err != nil {
		fmt.Printf("Error creating instance: %v\n", err)
		return
	}

	fmt.Printf("Created system container: %s (Status: %s)\n", req.Name, op.Get().Status)
}
```

## Interview Questions

**Q: What is the fundamental difference between LXC and Docker?**
**A:** LXC focuses on "System Containers," providing a full OS environment with an init system (PID 1 is systemd), suitable for long-running, multi-process workloads. Docker focuses on "Application Containers," packaging a single process per container, designed for ephemeral, stateless microservices.

**Q: How does LXC achieve isolation without a Hypervisor?**
**A:** LXC uses Linux Kernel features: **Namespaces** to isolate what a process sees (Network, PID, Mounts) and **Cgroups** to limit what a process can use (CPU, RAM). It shares the host's kernel, which is why it has near-native performance but cannot run a different OS kernel (e.g., you cannot run Windows on an LXC Linux host).

**Q: When would you prefer a System Container (LXC/Incus) over a Virtual Machine (KVM/ESXi)?**
**A:** When you need the behavioral isolation of a VM but want higher density and performance. LXC has zero hypervisor overhead and shares the host kernel, allowing you to run hundreds of containers on a single machine where you might only run dozens of VMs.

**Q: What is Incus, and why was it created?**
**A:** Incus is a community fork of LXD. It was created after Canonical moved LXD to the AGPLv3 license and relocated its development behind their corporate structure. Incus aims to provide a community-led, Apache-licensed alternative for managing system containers.

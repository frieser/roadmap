---
---

# LXC / LXD / Incus

**LXC (Linux Containers)** is an OS-level virtualization method for running multiple isolated Linux systems (containers) on a control host using a single Linux kernel. **LXD** (and its community fork **Incus**) is the next-generation system container manager that sits on top of LXC.

## 1. System vs. Application Containers

The distinction is crucial for understanding the ecosystem.

| Feature | Application Containers (Docker) | System Containers (LXC/Incus) |
| :--- | :--- | :--- |
| **Focus** | Single Process / Microservice | Full OS / Virtual Machine replacement |
| **Init Process** | Application binary is PID 1 | `systemd` or `init` is PID 1 |
| **Lifecycle** | Ephemeral, stateless | Persistent, long-running |
| **Management** | Declarative (Dockerfile) | Imperative (like managing a VM) |

## 2. Architecture

LXC provides a userspace interface for the Linux kernel features:
*   **Namespaces**: For isolation (PID, NET, MNT, etc.).
*   **Cgroups**: For resource limitation (CPU, RAM).
*   **AppArmor/Seccomp**: Mandatory Access Control policies to restrict what the container can do.

**Incus (The Fork)**:
Following Canonical's relicensing of LXD, the community created **Incus**. It is the modern, truly open-source daemon that manages LXC containers. It provides a REST API to create, start, stop, and migrate containers.

## 3. When to use LXC over Docker?

1.  **Legacy Applications**: Monolithic apps that expect a full OS environment (syslog, cron, sshd) and are hard to decouple into microservices.
2.  **Infrastructure Simulation**: Simulating a cluster of 100 servers (e.g., for testing Ansible playbooks) on a single laptop. LXC is much lighter than 100 VMs.
3.  **Pet vs. Cattle**: Docker is for "Cattle" (disposable). LXC is for "Pets" (servers you name and maintain).

## 4. Go Integration

Incus is written in Go and exposes a powerful Go client.

```go
package main

import (
	"fmt"
	"github.com/lxc/incus/client"
	"github.com/lxc/incus/shared/api"
)

func main() {
	// Connect to local Incus
	c, _ := incus.ConnectIncusUnix("", nil)

	// Define container
	req := api.InstancesPost{
		Name: "my-vps",
		Source: api.InstanceSource{Type: "image", Alias: "ubuntu/24.04"},
	}

	// Launch
	op, _ := c.CreateInstance(req)
	op.Wait()
	fmt.Println("System Container Launched!")
}
```

## 5. Interview Questions

**Q: How does LXC achieve isolation without a Hypervisor?**
**A:** By using Linux Namespaces. For example, the Network Namespace gives the container its own network stack, while the Mount Namespace gives it its own filesystem view. The kernel is shared, but the *view* of the resources is isolated.

**Q: Can you run Windows inside an LXC container?**
**A:** No. LXC relies on sharing the host's Linux kernel. It can only run distributions that are compatible with the host kernel (e.g., Debian on Ubuntu). To run Windows on Linux, you need a Hypervisor (VM).

**Q: What is the security trade-off of LXC vs VMs?**
**A:** LXC shares the kernel. A kernel vulnerability (e.g., in a syscall) can compromise the entire host. VMs have a hardware-enforced boundary (Hypervisor) which is much harder to penetrate.

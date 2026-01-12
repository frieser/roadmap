---
---

# LXC (Linux Containers)

Before Docker popularized "application containers," LXC (Linux Containers) introduced "system containers." LXC offers an environment that looks and feels like a lightweight Virtual Machine (VM) but shares the host's kernel, providing the performance of bare metal with the isolation of a VM.

## Summary

LXC is an operating-system-level virtualization method for running multiple isolated Linux systems (containers) on a control host using a single Linux kernel. Unlike Docker, which typically runs a single process per container (ephemeral), LXC containers run a full init system (Systemd/SysVinit), spawn multiple processes, and are meant to be treated like long-lived persistent servers (pet vs. cattle). **LXD** is the modern hypervisor/manager built on top of LXC to improve the user experience.

## Detailed Explanation

### 1. System Containers vs. Application Containers
*   **Docker (App Containers)**: Designed to package and run a *single application* (e.g., Nginx). They are ephemeral, stateless, and built from layers.
*   **LXC (System Containers)**: Designed to run a *full OS* (e.g., Ubuntu, Alpine). You SSH into them, install packages via `apt`, and manage services via `systemctl`. They persist data and state.

### 2. LXD (The Container Hypervisor)
LXD is a daemon that manages LXC containers. It provides:
*   **REST API**: For remote management.
*   **Image Server**: Like Docker Hub, but for full OS images.
*   **Storage Pools**: Advanced backend support (ZFS, Btrfs, LVM).
*   **Clustering**: Managing containers across multiple physical nodes.

### 3. Key Technologies
LXC relies on the same kernel features as Docker:
*   **Namespaces**: For isolation.
*   **Cgroups**: For resource limitation.
*   **AppArmor/SELinux**: For mandatory access control security profiles.

---

## Go Implementation Example

LXD is written in Go, and Canonical provides a robust client library (`github.com/canonical/lxd/client`) to manage containers programmatically.

```go
package main

import (
	"fmt"
	"log"

	lxd "github.com/canonical/lxd/client"
	"github.com/canonical/lxd/shared/api"
)

func main() {
	// 1. Connect to the local LXD daemon via Unix socket
	c, err := lxd.ConnectLXDUnix("", nil)
	if err != nil {
		log.Fatalf("Failed to connect to LXD: %v", err)
	}

	// 2. Define a new container request
	req := api.InstancesPost{
		Name: "my-go-container",
		Source: api.InstanceSource{
			Type:  "image",
			Alias: "ubuntu/22.04", // Use a remote image alias
		},
	}

	// 3. Create the container
	// This returns an Operation, as creation is async
	op, err := c.CreateInstance(req)
	if err != nil {
		log.Fatalf("Failed to create instance: %v", err)
	}
	
	// Wait for the operation to complete
	err = op.Wait()
	if err != nil {
		log.Fatalf("Creation operation failed: %v", err)
	}

	// 4. Start the container
	reqState := api.InstanceStatePut{
		Action: "start",
		Timeout: -1,
	}
	op, err = c.UpdateInstanceState("my-go-container", reqState, "")
	if err != nil {
		log.Fatal(err)
	}
	op.Wait()

	fmt.Println("LXC Container 'my-go-container' created and started!")
}
```

## Interview Questions

**Q: When would you choose LXC/LXD over Docker?**
**A:** Use LXC/LXD when you need a "lightweight VM" experience. Examples include:
1.  Running legacy applications that expect a full OS environment (init system, cron, syslog).
2.  Simulating a cluster of servers on a single developer machine (e.g., testing Ansible playbooks against 3 "servers").
3.  Providing isolated shell environments to users (e.g., a VPS provider).

**Q: Can LXC containers run Docker?**
**A:** Yes, this is known as "Docker-in-LXC" (or Docker-in-LXD). Since an LXC container is a full OS, you can install the Docker daemon inside it. However, this often requires configuring the LXC container to be "privileged" or enabling "nesting" security profiles to allow the inner Docker daemon to manipulate namespaces.

**Q: How does LXC handle storage compared to Docker?**
**A:** Docker uses a layered filesystem (OverlayFS) where changes are written to a diff layer. LXC typically treats the container's filesystem as a standard directory tree or a subvolume (on ZFS/Btrfs). While LXC supports snapshots, it doesn't use the "build layer" concept of Dockerfiles; updates are applied via package managers (`apt`, `yum`) inside the running container.

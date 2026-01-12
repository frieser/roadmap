---
---

# Docker

Docker is a platform for developing, shipping, and running applications in containers. It is the de-facto standard for containerization in the industry.

## 1. Internals: The Linux Foundation

Docker is not a virtualization technology; it is a process isolation technology. It relies on three core Linux kernel features:

### Namespaces (Isolation)
Namespaces provide the "view" of the system. They make a process think it has its own independent system resources.
*   **PID Namespace**: The container sees its process as PID 1.
*   **NET Namespace**: Own network stack (IP, ports, routes).
*   **MNT Namespace**: Own filesystem mount points.
*   **UTS Namespace**: Own hostname.
*   **IPC Namespace**: Own inter-process communication channels.
*   **USER Namespace**: Own user/group IDs (allows mapping root in container to non-root on host).

### Cgroups (Resource Control)
Control Groups (cgroups) limit and measure the resources a container can use.
*   **Limits**: Max CPU shares, memory limits (`--memory=512m`), block I/O.
*   **Prioritization**: Giving some containers higher priority for CPU time.
*   **Accounting**: Tracking how much resource was consumed.

### UnionFS (Storage)
Docker uses a layered filesystem (typically **Overlay2**).
*   **Layers**: Each command in a `Dockerfile` creates a read-only layer.
*   **Copy-on-Write (CoW)**: When a container modifies a file, it copies the file from the read-only lower layer to the writable upper layer. The original file remains unchanged.

## 2. Networking Models

*   **Bridge (Default)**: Containers are connected to a virtual bridge (`docker0`). They get an IP address on an internal subnet and communicate via NAT.
*   **Host**: The container shares the host's networking namespace. No NAT. Highest performance but port conflicts are possible.
*   **Overlay**: Used for multi-host communication (Docker Swarm/K8s). Encapsulates traffic in VXLAN.
*   **Macvlan**: Assigns a MAC address to the container, making it appear as a physical device on the network.

## 3. Production Best Practices

### Multi-Stage Builds
Drastically reduce image size by separating the build environment from the runtime environment.

```dockerfile
# Build Stage
FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

# Runtime Stage
FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

### Security Hardening
*   **Distroless Images**: Use images like `gcr.io/distroless/static` which contain only the app and runtime dependencies (no shell, no package manager).
*   **Rootless Mode**: Run the Docker daemon as a non-root user to mitigate privilege escalation attacks.
*   **Read-Only Filesystem**: Run containers with `--read-only` to prevent attackers from writing payloads.

## 4. Go Integration (Docker SDK)

Programmatically managing containers using `github.com/docker/docker/client`.

```go
package main

import (
	"context"
	"fmt"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
)

func main() {
	ctx := context.Background()
	cli, _ := client.NewClientWithOpts(client.FromEnv)

	// Create a container
	resp, _ := cli.ContainerCreate(ctx, 
        &container.Config{Image: "alpine", Cmd: []string{"echo", "hello world"}}, 
        nil, nil, nil, "")

	// Start it
	cli.ContainerStart(ctx, resp.ID, container.StartOptions{})
	fmt.Println("Container ID:", resp.ID)
}
```

## 5. Interview Questions

**Q: What is the "Zombie Process" problem in Docker?**
**A:** If the entrypoint (PID 1) is a shell script or app that doesn't handle process reaping, child processes that die become "zombies" (defunct) and consume system resources. **Solution**: Use `tini` (`--init`) or a proper process manager as the entrypoint.

**Q: Why use Distroless images over Alpine?**
**A:** Alpine is small but still contains a shell (`/bin/sh`) and package manager (`apk`). Distroless images have *no* shell, making it much harder for an attacker to execute commands or install tools if they compromise the application (RCE).

**Q: Explain Overlay2's "Whiteout File".**
**A:** When you delete a file in a Docker container, it isn't physically removed from the read-only image layers. Instead, a "whiteout" file (a special character device) is created in the writable layer to mask the file's existence. The file still occupies disk space in the lower layers.

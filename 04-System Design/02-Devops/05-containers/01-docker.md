---
---

# Docker for DevOps

Docker revolutionized the software industry by popularizing containers—lightweight, standalone, and executable software packages that include everything needed to run an application. For DevOps, Docker is the fundamental unit of deployment, ensuring consistency across development, testing, and production environments ("Build Once, Run Anywhere").

## Summary

Docker simplifies application delivery by wrapping code, runtime, and dependencies into a standardized unit called a **container**. It relies on Linux kernel features like **Namespaces** (isolation) and **Cgroups** (resource limits). Key concepts include **Images** (read-only templates), **Containers** (runnable instances), and **Dockerfiles** (build instructions). Mastering Docker involves understanding **multi-stage builds** for optimizing image size and managing **networks** and **volumes** for communication and persistence.

## Detailed Explanation

### 1. Docker Internals
*   **Namespaces**: Provide isolation.
    *   `PID`: Process isolation (containers can't see host processes).
    *   `NET`: Network isolation (own IP, ports).
    *   `MNT`: Filesystem isolation.
*   **Cgroups (Control Groups)**: Manage resource usage.
    *   Limit CPU, Memory, and Disk I/O to prevent one container from hogging system resources.
*   **UnionFS (OverlayFS)**: A layered filesystem.
    *   Images are built from read-only layers.
    *   When a container starts, a thin "writable layer" is added on top. This makes container creation instant and storage-efficient.

### 2. Best Practices
*   **Multi-Stage Builds**: Use a large image with build tools (e.g., `golang:1.21`) to compile the app, then copy only the binary to a tiny runtime image (e.g., `alpine` or `scratch`).
    ```dockerfile
    # Build Stage
    FROM golang:1.21 AS builder
    WORKDIR /app
    COPY . .
    RUN go build -o myapp

    # Runtime Stage
    FROM alpine:latest
    COPY --from=builder /app/myapp /myapp
    CMD ["/myapp"]
    ```
*   **Rootless Mode**: Running the Docker daemon and containers as a non-root user to mitigate security risks (container breakouts).

### 3. Docker SDK for Go
Since Docker itself is written in Go, it offers a first-class SDK (`github.com/docker/docker/client`) to control the daemon programmatically. This is used by tools like Terraform, CI agents, and custom orchestrators.

---

## Go Implementation Example

This example demonstrates how to use the Docker SDK to list running containers and inspect their state—a common task for monitoring agents.

```go
package main

import (
	"context"
	"fmt"
	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
)

func main() {
	// 1. Initialize Docker Client
	// NegotiateAPIVersion ensures compatibility with the local daemon
	cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
	if err != nil {
		panic(err)
	}

	// 2. List Containers (Equivalent to 'docker ps')
	containers, err := cli.ContainerList(context.Background(), container.ListOptions{})
	if err != nil {
		panic(err)
	}

	fmt.Println("Running Containers:")
	for _, container := range containers {
		fmt.Printf("ID: %s | Image: %s | State: %s\n", container.ID[:10], container.Image, container.State)
	}

	// 3. Inspect a specific container (Conceptual)
	// inspection, err := cli.ContainerInspect(context.Background(), containers[0].ID)
}
```

### Implementing a "Container" from Scratch
To truly understand Docker, it helps to know how to create a container using raw Linux syscalls in Go. This involves setting up namespaces using `cmd.SysProcAttr`.

```go
// +build linux

package main

import (
	"fmt"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	switch os.Args[1] {
	case "run":
		run()
	case "child":
		child()
	default:
		panic("help")
	}
}

func run() {
	// Re-execute this binary with "child" argument
	cmd := exec.Command("/proc/self/exe", append([]string{"child"}, os.Args[2:]...)...)
	
	// Enable Namespaces: UTS (Hostname), PID (Process IDs), MNT (Mounts)
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Printf("Error: %v\n", err)
		os.Exit(1)
	}
}

func child() {
	// We are now inside the namespace!
	fmt.Printf("Running %v as PID %d\n", os.Args[2:], os.Getpid())

	// Change hostname inside container
	syscall.Sethostname([]byte("container-demo"))

	// Execute the user command (e.g., /bin/bash)
	cmd := exec.Command(os.Args[2], os.Args[3:]...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	cmd.Run()
}
```

## Interview Questions

**Q: What is the difference between an Image and a Container?**
**A:** An **Image** is an immutable, read-only template that contains the application code, libraries, and dependencies. A **Container** is a running instance of an image. It adds a writable layer on top of the image, allowing the application to modify files during execution without affecting the underlying image.

**Q: Explain the PID 1 problem in Docker.**
**A:** In a Linux system, PID 1 (init) has special responsibilities, like reaping zombie processes and handling signals (SIGTERM). If a container's entrypoint (e.g., a shell script or Java app) doesn't handle these tasks correctly, zombie processes can accumulate, or the container might not shut down gracefully. Solutions include using `tini` (init for containers) or ensuring the app handles signals.

**Q: How do `ENTRYPOINT` and `CMD` differ?**
**A:**
*   `ENTRYPOINT`: The command that *always* runs when the container starts. It makes the container behave like an executable.
*   `CMD`: Provides *default arguments* to the ENTRYPOINT. If the user specifies arguments `docker run image arg1`, they override `CMD` but are passed to `ENTRYPOINT`.

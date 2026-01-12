#Linux
---
tags: ['linux', 'roadmap', 'tools']
---

## Summary
A **Container Runtime** is the software responsible for running containers. The ecosystem is split into **low-level runtimes** (like `runc`), which handle the heavy lifting of interacting with the Linux kernel (namespaces, cgroups), and **high-level runtimes** (like `containerd` and `CRI-O`), which manage images, storage, and networking while delegating the execution to low-level runtimes. This modularity is governed by the **Open Container Initiative (OCI)** standards.

## Detailed Explanation

### The OCI Standards
The **Open Container Initiative (OCI)** was established to ensure that containers are portable and interoperable. It defines two key specifications:
1. **Runtime Specification**: Defines how to run a "filesystem bundle" as a container.
2. **Image Specification**: Defines how to create and store container images.

### 1. Low-Level Runtimes: `runc`
`runc` is the reference implementation of the OCI Runtime Spec. It is a CLI tool for spawning and running containers on Linux according to the OCI specification. It performs the actual syscalls to create namespaces and cgroups.

#### **Bash Example: Running a container with `runc`**
To run a container with `runc`, you need an OCI bundle (a directory with a `config.json` and a root filesystem).

```bash
# 1. Create a rootfs directory
mkdir -p my-container/rootfs

# 2. Export a docker image into the rootfs
# (Assuming docker is installed)
docker export $(docker create alpine) | tar -C my-container/rootfs -xvf -

# 3. Generate the default OCI spec (config.json)
cd my-container
runc spec

# 4. Run the container
sudo runc run my-container-id
```

### 2. High-Level Runtimes: `containerd` and `CRI-O`
High-level runtimes provide the management layer. They handle image pulling, storage management, networking setup, and process monitoring.

#### **containerd**
Originally part of Docker, `containerd` is now a standalone CNCF graduated project. It is widely used in production environments and as the default runtime for many Kubernetes distributions.

#### **CRI-O**
An implementation of the Kubernetes **Container Runtime Interface (CRI)**. Unlike `containerd`, which supports multiple clients, `CRI-O` is purpose-built specifically for Kubernetes, making it highly lightweight and optimized for K8s clusters.

### 3. The Container Runtime Interface (CRI)
The **CRI** is a plugin interface which enables the `kubelet` (the K8s agent) to use a wide variety of container runtimes, without having to recompile the cluster components.

### **Bash Example: Interacting with containerd via `ctr`**
`containerd` comes with a CLI tool called `ctr` for debugging and management.

```bash
# Pull an image
sudo ctr images pull docker.io/library/alpine:latest

# Run a container
sudo ctr run --rm docker.io/library/alpine:latest my-alpine-container sh -c "echo Hello from containerd"

# List running containers
sudo ctr containers list
```

## Interview Questions

**Q: What is the difference between a high-level and a low-level container runtime?**
**A:** A low-level runtime (e.g., `runc`) focuses solely on running the container by interacting with kernel features like namespaces and cgroups. A high-level runtime (e.g., `containerd`, `CRI-O`) manages the higher-level concerns like image management, storage, networking, and the lifecycle of multiple containers, typically delegating the actual execution to a low-level runtime.

**Q: Why was the OCI (Open Container Initiative) created?**
**A:** It was created to prevent fragmentation in the container industry by providing a set of open, industry-standard specifications for container formats and runtimes, ensuring that a container built with one tool can run on any other compliant runtime.

**Q: How does `containerd` relate to Docker?**
**A:** `containerd` was originally the core container runtime inside Docker. Docker was later refactored to use `containerd` as its internal runtime component. Eventually, `containerd` was donated to the CNCF as a standalone project, and it can now be used independently of Docker, notably as a Kubernetes runtime.

**Q: What is the role of `shim` (e.g., containerd-shim)?**
**A:** The shim acts as an intermediary between the high-level runtime and the low-level runtime. It allows the runtime (like `containerd`) to exit or be upgraded without stopping the containers, handles the terminal (I/O) of the container, and reports the exit status of the container back to the high-level runtime.

**Q: When would you choose `CRI-O` over `containerd`?**
**A:** `CRI-O` is a better choice when you want a minimal, purpose-built runtime strictly for Kubernetes. It has a smaller footprint and is easier to secure because it only implements what Kubernetes needs. `containerd` is more versatile and can be used for non-Kubernetes workloads as well.

---
---

# Containerization vs. Virtualization

Understanding the trade-offs between Virtual Machines (VMs) and Containers is fundamental for system design. It is a choice between **Isolation** and **Efficiency**.

## 1. Core Differences

| Feature | Virtual Machine (VM) | Container |
| :--- | :--- | :--- |
| **Abstraction** | Hardware (Hypervisor) | OS Kernel (Engine) |
| **Kernel** | Own Guest Kernel | Shares Host Kernel |
| **Boot Time** | Minutes | Seconds / Milliseconds |
| **Size** | GBs (Full OS) | MBs (App + Libs) |
| **Isolation** | Strong (Hardware boundary) | Weaker (Process boundary) |

## 2. Isolation & Security

### The "Kernel Panic" Radius
*   **VM**: If the Guest Kernel panics, only that VM crashes. The Host and other VMs are unaffected.
*   **Container**: If the Shared Host Kernel panics, **ALL** containers on that host crash.

### Attack Surface
*   **Container Escape**: Because the kernel is shared, a vulnerability in a syscall can allow an attacker to "escape" the container and gain root on the host.
*   **Mitigation**:
    *   **Seccomp**: Whitelist only necessary syscalls.
    *   **User Namespaces**: Map `root` inside container to `nobody` outside.
    *   **gVisor / Kata Containers**: Provide a sandbox (userspace kernel or microVM) for untrusted workloads.

## 3. Modern Trends: The Convergence

The line is blurring with "Sandboxed Containers".

*   **Firecracker (MicroVMs)**: Used by AWS Lambda. Runs container-like workloads inside stripped-down KVM virtual machines (~125ms boot time). Combines VM isolation with Container speed.
*   **gVisor**: Used by Google Cloud Run. Intercepts syscalls in userspace to prevent direct kernel access.

## 4. Performance

*   **Containers**: Near-native performance. CPU/RAM overhead is negligible.
*   **VMs**: Hypervisor overhead (CPU instruction translation, I/O emulation).

## 5. Interview Questions

**Q: Why are containers lighter than VMs?**
**A:** Containers do not boot a full Operating System. They are simply isolated processes running on the host kernel. They don't need to initialize hardware drivers, init systems, or duplicate kernel memory.

**Q: When would you strictly choose a VM over a Container?**
**A:**
1.  **Strong Isolation**: Running untrusted code (e.g., multi-tenant SaaS where customers run arbitrary code).
2.  **Kernel Requirements**: The app needs a specific kernel version or kernel modules that the host doesn't have.
3.  **OS Diversity**: You need to run Windows apps on a Linux host (requires hardware virtualization).

**Q: What is a "Scratch" container?**
**A:** A container that contains *only* the application binary and nothing else (no shell, no libs). It is the ultimate minimal attack surface, commonly used with statically linked Go applications.

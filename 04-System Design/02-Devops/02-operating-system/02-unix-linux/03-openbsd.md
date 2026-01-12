---
---

# OpenBSD for DevOps

OpenBSD is a security-focused Unix-like operating system that emphasizes proactive mitigation, code quality, and a "secure by default" philosophy. In a DevOps context, it is a premier choice for high-security infrastructure components.

## Summary

OpenBSD is renowned for its focus on security, having had "only two remote holes in the default install, in a heck of a long time." It provides unique security primitives like `pledge` and `unveil` that allow developers to implement fine-grained sandboxing with minimal code changes. Its stateful firewall, `pf`, is considered one of the most powerful and easy-to-configure firewalls available.

## Detailed Development

### 1. PF (Packet Filter)
`pf` is OpenBSD's system for filtering TCP/IP traffic and doing Network Address Translation (NAT).
- **Security First**: Integrated directly into the kernel for performance and security.
- **Human-Readable Syntax**: Highly praised for its logical and clean configuration (`/etc/pf.conf`).
- **Features**: Stateful inspection, NAT, QoS (ALTQ), and integration with `relayd` for load balancing.
- **Redundancy**: Works with `CARP` (Common Address Redundancy Protocol) for high-availability firewall clusters.

### 2. Pledge
`pledge(2)` is a system call that allows a process to voluntarily restrict the system calls it can make.
- **Concept**: A program "promises" to only use specific capabilities (e.g., `stdio`, `rpath`, `inet`).
- **Enforcement**: If the process attempts a syscall outside its promises, the kernel terminates it immediately (`SIGABRT`).
- **DevOps Use Case**: Hardening custom tools and agents against exploitation by limiting their attack surface.

### 3. Unveil
`unveil(2)` restricts a process's view of the filesystem.
- **Concept**: A program specifies exactly which files or directories it needs to access and with what permissions (`r`, `w`, `x`, `c`).
- **Enforcement**: The rest of the filesystem becomes invisible to the process. Unlike `pledge`, violations usually return `ENOENT` (not found), allowing for graceful error handling.
- **DevOps Use Case**: Ensuring that a web server or CI agent can only see its specific workspace and configuration files.

## Go Support and Security Interaction

Go has first-class support for OpenBSD's security features through the `golang.org/x/sys/unix` package.

### Implementation Example
```go
package main

import (
	"fmt"
	"log"
	"golang.org/x/sys/unix"
)

func main() {
	// 1. Unveil only the necessary directories
	// Allow reading from /tmp and current directory
	if err := unix.Unveil("/tmp", "rw"); err != nil {
		log.Fatal(err)
	}
	// Lock unveil (no more unveil calls allowed)
	if err := unix.UnveilBlock(); err != nil {
		log.Fatal(err)
	}

	// 2. Pledge to only use stdio and rpath/wpath
	if err := unix.Pledge("stdio rpath wpath", ""); err != nil {
		log.Fatal(err)
	}

	fmt.Println("Process successfully pledged and unveiled.")
}
```

### Security Considerations for Go
- **Runtime Threads**: Go's runtime creates several threads and performs initial setup. `pledge` should ideally be called as early as possible in `main()`.
- **Static vs. Dynamic**: OpenBSD prefers dynamic linking to ensure syscalls pass through `libc` (for security verification). Go has evolved to support this natively on OpenBSD.
- **Init Functions**: Code in `init()` functions runs before `main()`. If an `init()` function performs a restricted action (like network access) before `main()` calls `pledge`, it will succeed, but once `pledge` is active, that action will be blocked.

## Interview Preparation Questions

1. **What is the difference between `pledge` and `unveil`?**
   - *Answer*: `pledge` restricts the *types of operations* (syscalls) a process can perform, while `unveil` restricts the *filesystem paths* a process can access.

2. **Explain the "Security by Default" philosophy in OpenBSD.**
   - *Answer*: It means that in a default installation, all non-essential services are disabled, and those that are enabled are running with the highest possible security mitigations (ASLR, W^X, `pledge`, etc.) without requiring user configuration.

3. **How does `pf` compare to Linux's `iptables` or `nftables` in terms of configuration?**
   - *Answer*: `pf` uses a more intuitive, centralized configuration file (`pf.conf`) that focuses on high-level rules and stateful inspection, whereas `iptables` is often seen as more imperative and rule-order sensitive.

4. **Why would a DevOps engineer choose OpenBSD over Linux for a VPN gateway?**
   - *Answer*: Due to its extreme focus on network security, the robustness of `pf`, and the lower probability of zero-day exploits in the base system compared to more complex kernels.

5. **How does `pledge` implement the Principle of Least Privilege?**
   - *Answer*: It allows a developer to programmatically strip away any privileges or capabilities that a process does not strictly need after its initialization phase.

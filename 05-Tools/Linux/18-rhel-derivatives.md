---
title: RHEL Derivatives
tags: [linux, rhel, devops, roadmap]
---

# RHEL and Derivatives (CentOS, Fedora, Rocky, Alma)

## Summary
Red Hat Enterprise Linux (RHEL) is the industry standard for enterprise-grade Linux, known for its long-term support and stability. The RHEL ecosystem operates on a tiered lifecycle where features move from bleeding-edge community projects (Fedora) to development streams (CentOS Stream) before reaching production-ready releases (RHEL) and their downstream binary-compatible clones (Rocky Linux, AlmaLinux).

## Detailed Explanation

### 1. The RHEL Ecosystem Hierarchy
The relationship between these distributions is defined by their position in the development pipeline and their intended stability.

| Tier | Distribution | Role | Stability | Relation to RHEL |
| :--- | :--- | :--- | :--- | :--- |
| **Upstream** | **Fedora** | Innovation Hub | Low (Bleeding Edge) | Experimental features eventually land in RHEL. |
| **Midstream** | **CentOS Stream** | Development | Medium (Rolling) | A preview of the next RHEL minor release. |
| **Enterprise**| **RHEL** | Production | High (Certified) | Commercial product with paid support. |
| **Downstream**| **Rocky Linux** | 1:1 Rebuild | High (Stable) | Community-led "bug-for-bug" compatible clone. |
| **Downstream**| **AlmaLinux** | ABI Compatible | High (Stable) | Community-led binary-compatible clone. |

#### **Upstream vs Downstream Evolution**
Historically, CentOS was a downstream rebuild of RHEL. However, in 2020, Red Hat shifted **CentOS Stream** to be "upstream" of RHEL. This led to the creation of **Rocky Linux** and **AlmaLinux** to fill the void for users needing a free, downstream, stable production environment compatible with RHEL.

---

### 2. The RPM Ecosystem
RHEL-based systems use the **RPM (Red Hat Package Manager)** format. Management is split into low-level and high-level tools.

#### **RPM (Low-Level Tool)**
The `rpm` command manages individual `.rpm` files but does not handle dependency resolution.
```bash
# Install a local rpm file
rpm -ivh package-1.0.el9.x86_64.rpm

# Query if a package is installed
rpm -q package_name

# List all files installed by a package
rpm -ql package_name
```

#### **DNF / YUM (High-Level Tools)**
**DNF (Dandified YUM)** is the modern package manager (default since RHEL 8). It handles complex dependency resolution using the `libsolv` library.
```bash
# Install a package from configured repositories
sudo dnf install httpd

# Search for a package
dnf search nginx

# Find which package provides a specific file/binary
dnf provides /usr/bin/nslookup

# Manage groups of packages
sudo dnf groupinstall "Development Tools"
```
*Note: `yum` is typically a symbolic link to `dnf` in modern versions for backward compatibility.*

---

### 3. Enterprise Stability Tiers & Features
RHEL derivatives are chosen for specific production-grade features:

- **SELinux (Security-Enhanced Linux):** A Mandatory Access Control (MAC) system that provides fine-grained security policies.
  - `getenforce`: Check current mode (Enforcing, Permissive, Disabled).
  - `ls -Z`: View security contexts of files.
- **AppStream:** Allows multiple versions of software (e.g., Python 3.9 and 3.11) to exist on the same system via modularity.
- **Long Lifecycle:** RHEL versions typically receive 10 years of support, ensuring environment consistency for large-scale deployments.

---

## Interview Questions

### 1. What is the difference between CentOS Stream and the old CentOS Linux?
**Answer:** CentOS Linux was a downstream rebuild of RHEL (released after RHEL). CentOS Stream is an upstream development branch (released before RHEL), serving as a rolling preview of what will become the next minor version of RHEL.

### 2. How does DNF differ from RPM?
**Answer:** `rpm` is a low-level tool that works with individual package files and cannot resolve dependencies. `dnf` (and its predecessor `yum`) is a high-level tool that interacts with repositories and automatically resolves and installs all necessary dependencies for a package.

### 3. A file has 777 permissions but a service still gets "Permission Denied". What should you check?
**Answer:** You should check the **SELinux** context. Even with standard Unix permissions allowed, SELinux policies might block the process's access to that file label. Use `ls -Z` to check contexts and `ausearch -m AVC` to check audit logs.

### 4. What command would you use to find out which package contains the `ifconfig` binary?
**Answer:** `dnf provides */ifconfig` or `dnf provides ifconfig`.

### 5. Why would a company choose AlmaLinux or Rocky Linux over Fedora for a production server?
**Answer:** Fedora is "bleeding-edge" with frequent updates and a short lifecycle (approx. 13 months), making it unstable for long-term production. AlmaLinux and Rocky Linux provide 10 years of binary-compatible stability based on RHEL, making them suitable for enterprise workloads that require consistency.

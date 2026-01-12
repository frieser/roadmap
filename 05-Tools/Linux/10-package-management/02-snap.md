---
tags: ['linux', 'roadmap']
---

# Snap Packages (snap install, classic confinement)

## Summary
Snap is a software packaging and deployment system developed by Canonical. It uses containerized packages called "snaps" that bundle all dependencies, ensuring they work across different Linux distributions regardless of underlying library versions.

## Detailed Explanation

### What are Snap Packages?
Snaps are self-contained applications that run in a sandbox with mediated access to the host system. They are designed to be secure, easy to update, and cross-platform. Unlike traditional package managers (APT, RPM), Snaps include all the libraries and runtimes they need to function.

### Containerization and SquashFS
Every snap is a compressed **SquashFS** filesystem image. When you install or run a snap, this image is mounted as a read-only loop device at `/snap/<name>/<version>`. This ensures that the application files cannot be tampered with and that different versions of the same app can coexist.

### Confinement Modes
Security in Snap is handled through confinement, which uses AppArmor and Seccomp to restrict what the application can do:
- **Strict Confinement**: The default mode. The application is isolated from the system and can only access its own data or specific "interfaces" (like the network or home directory) if permitted.
- **Classic Confinement**: Used for applications that need full access to the system, such as compilers (GCC), IDEs (VS Code), or terminal emulators. These snaps are not sandboxed. 
    - *Installation Command*: `sudo snap install <name> --classic`
- **Devmode**: A "broken" confinement mode used by developers to see what permissions an app would need before releasing it with strict confinement.

### Key Features
- **Delta Updates**: Snap only downloads the changes between versions, saving bandwidth.
- **Automatic Rollbacks**: If an update fails to start, Snap automatically reverts to the previous working version.
- **Channels**: Users can track different release streams:
    - `stable`: Tested, reliable versions.
    - `candidate`: For users helping with final testing.
    - `beta`: For testing upcoming features.
    - `edge`: Latest builds, potentially unstable.

### Comparison: Snap vs. APT
| Feature | Snap | APT (Traditional) |
| :--- | :--- | :--- |
| **Dependencies** | Bundled inside the package | Shared with the system |
| **Isolation** | Sandboxed by default | Full system access |
| **Updates** | Automatic & Atomic | Manual/System-wide |
| **File System** | Mounted SquashFS (Read-only) | Unpacked into `/usr`, `/bin`, etc. |
| **Startup Speed** | Slower (mounting overhead) | Faster (native execution) |

### Common Bash Commands
```bash
# Search for a package
snap find spotify

# Install a strictly confined package
sudo snap install spotify

# Install a package with classic confinement
sudo snap install code --classic

# List installed snaps and their versions
snap list

# Update all snaps (happens automatically 4x a day)
sudo snap refresh

# Switch to a different channel
sudo snap refresh vlc --channel=beta

# Revert to the previously installed version
sudo snap revert vlc

# Remove a snap and its user data
sudo snap remove vlc
```

## Interview Questions

### 1. What does the `--classic` flag do when installing a snap?
**Answer:** It disables the default security sandbox (confinement). This is required for tools that need to access the host's root filesystem, system libraries, or compilers, which would otherwise be blocked by strict confinement.

### 2. How does Snap solve the "Dependency Hell" problem found in traditional Linux distributions?
**Answer:** Snap bundles every dependency required by the application (libraries, runtimes, etc.) into a single, immutable SquashFS image. This ensures the app runs exactly the same way regardless of which libraries are installed on the host system.

### 3. Explain the mounting mechanism used by Snap.
**Answer:** When a snap is installed, its SquashFS image is mounted as a read-only filesystem under `/snap/package-name/revision`. The `snapd` daemon manages these mounts, ensuring that the application environment is consistent and isolated.

### 4. What happens if a Snap update causes the application to fail?
**Answer:** Snap supports atomic updates and automatic rollbacks. If a new version fails to initialize or run correctly, the `snapd` service can automatically revert the system to the previous working revision, preserving the application's state.

### 5. Why might a developer choose Snap over a native `.deb` or `.rpm` package?
**Answer:** Developers can target multiple Linux distributions (Ubuntu, Fedora, Arch, etc.) with a single package. They also gain control over the update cycle and can ensure users are always running the latest version without waiting for distribution maintainers to update their repositories.

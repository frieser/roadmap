---
---

## Summary
Package managers automate the process of installing, upgrading, configuring, and removing computer programs for a computer's operating system in a consistent manner. `apt` (Debian/Ubuntu) and `yum` (RHEL/CentOS) are the two most common standard package managers in the Linux world.

## Detailed Explanation

### APT (Advanced Package Tool) - Debian Family
*   **Update Repo List**: `apt update` (Crucial before install).
*   **Install**: `apt install nginx`.
*   **Upgrade All**: `apt upgrade`.
*   **Remove**: `apt remove nginx` (Keep config), `apt purge nginx` (Delete config).
*   **Search**: `apt search nginx`.

### YUM (Yellowdog Updater, Modified) - RHEL Family
*   **Update Repo List**: `yum check-update` (Often implicit).
*   **Install**: `yum install nginx`.
*   **Upgrade All**: `yum update`.
*   **Remove**: `yum remove nginx`.
*   **Search**: `yum search nginx`.

### Key Differences
`apt` separates "update list" (`update`) from "upgrade packages" (`upgrade`). `yum update` typically does both (refreshes metadata and upgrades).

## Go-Specific Context/Examples

Go binaries are often distributed as source (go install) or tarballs, but for production, you package them as `.deb` or `.rpm`.

### Example: Installing Go with Apt
```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go
```

## Interview Questions

**Q: What is the difference between `apt` and `apt-get`?**
**A:** `apt` is a newer, user-friendly wrapper designed for interactive usage (has progress bars, colors). `apt-get` is the low-level backend tool, preferred for scripting (stable output).

**Q: What does `yum clean all` do?**
**A:** It clears the local cache of package metadata and headers. This is useful when you are getting errors about mismatched checksums or if the repo data is stale.

**Q: How do you fix broken dependencies in apt?**
**A:** `apt --fix-broken install` (or `apt-get -f install`). This attempts to correct a system with unmet dependencies.

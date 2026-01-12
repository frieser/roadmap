---
tags: ['linux', 'roadmap']
---

# System Logs (dmesg, boot.log, syslog)

## Summary
System logs are the primary diagnostic tool in Linux for monitoring system health and troubleshooting issues. They range from low-level kernel messages captured in the **kernel ring buffer** (`dmesg`) to high-level application and service events recorded in general system logs like `/var/log/syslog` or the systemd journal. Understanding where these logs reside and how to query them is critical for debugging boot failures and hardware issues.

## Detailed Explanation

### 1. Kernel Ring Buffer (`dmesg`)
The kernel has its own logging buffer that starts collecting messages the moment the kernel is loaded, even before the root filesystem is mounted.

*   **What it is**: A circular buffer in memory (ring buffer) where the kernel writes its messages. Once full, older messages are overwritten.
*   **Usage**: Access it using the `dmesg` command.
*   **Key Flags**:
    *   `dmesg -T`: Display human-readable timestamps.
    *   `dmesg -l err,warn`: Filter by log levels (e.g., only show errors and warnings).
    *   `dmesg -w`: Follow the log in real-time (similar to `tail -f`).

```bash
# View last 20 kernel messages with human-readable timestamps
dmesg -T | tail -20

# Search for specific hardware issues (e.g., USB or SATA)
dmesg | grep -i "usb"
```

### 2. Boot Logs (`/var/log/boot.log`)
As the system transitions from the kernel initialization to user-space (init/systemd), service startup messages are often captured here.

*   **Source**: In systemd-based systems, `systemd-journald` handles this, but some distributions still write to `/var/log/boot.log` via a specific service.
*   **Content**: Success or failure indicators ([ OK ] or [ FAILED ]) for services like networking, SSH, and disk mounting.

```bash
# Check the boot log for failed services
grep "FAILED" /var/log/boot.log
```

### 3. System Logs (`/var/log/syslog` and `/var/log/messages`)
Traditional syslog daemons (`rsyslogd`, `syslog-ng`) collect logs from various system components and write them to text files.

*   **Location**: 
    *   Debian/Ubuntu: `/var/log/syslog`
    *   RHEL/CentOS/Fedora: `/var/log/messages`
*   **Content**: A combination of kernel logs, authentication logs, and service-specific logs.

```bash
# Monitor the system log in real-time
tail -f /var/log/syslog
```

### 4. The Modern Approach: `journalctl`
Most modern Linux distributions use `systemd-journald`, which stores logs in a binary format for faster querying and metadata support.

*   **Current Boot**: `journalctl -b`
*   **Specific Service**: `journalctl -u ssh.service`
*   **Filtering by Time**: `journalctl --since "1 hour ago"`

```bash
# View all logs from the current boot with priority 'error' or higher
journalctl -b -p err
```

### 5. Troubleshooting Boot Issues
1.  **Check `dmesg`**: Look for hardware initialization failures or driver crashes.
2.  **Check `journalctl -xb`**: The `-x` flag provides explanatory text for errors, and `-b` limits it to the current (failed) boot.
3.  **Check `/var/log/boot.log`**: Verify if a critical service (like the filesystem mount) failed.

## Interview Questions

**Q: What is the difference between `dmesg` and `journalctl`?**
**A:** `dmesg` specifically reads from the kernel ring buffer and contains only kernel-level messages. `journalctl` queries the systemd journal, which contains kernel logs (by reading from `/dev/kmsg`), service logs, and application logs in a unified, searchable binary format.

**Q: How do you view kernel logs that occurred *before* the current boot?**
**A:** If persistent logging is enabled in systemd, you can use `journalctl -b -1` to view logs from the previous boot. Standard `dmesg` only shows the current kernel buffer and is lost on reboot unless saved to a file.

**Q: Why might `/var/log/syslog` be empty on a modern system?**
**A:** Some "minimal" or modern distributions rely solely on `systemd-journald` and do not install a traditional syslog daemon (like `rsyslog`). In such cases, all logs are accessed via `journalctl`.

**Q: How can you find out which hardware device is causing a boot delay?**
**A:** Use `dmesg -d` to see the time delta between messages, or `systemd-analyze blame` to see which services took the longest to start during the boot process.

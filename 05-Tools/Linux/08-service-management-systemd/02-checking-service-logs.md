#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
`journalctl` is the primary command-line utility for querying and displaying logs from `journald`, the systemd logging service. Unlike traditional text-based logs (like `/var/log/syslog`), the journal is a binary format that stores metadata (service name, priority, timestamp, etc.) alongside the message, allowing for fast, structured, and flexible filtering of system logs.

## Detailed Explanation

### **Core Concepts**
The `systemd-journald` daemon collects logs from various sources: kernel, boot process, stdout/stderr from services, and syslog messages. These are indexed for performance.

### **Basic Viewing**
To view all logs (starts from the oldest):
```bash
journalctl
```

To see only the most recent logs (jumps to the end):
```bash
journalctl -e
```

### **Filtering by Service (Unit)**
The most common task is checking logs for a specific service:
```bash
# View all logs for Nginx
journalctl -u nginx.service
```

### **Real-time Monitoring**
Similar to `tail -f`, you can "follow" logs as they occur:
```bash
# Follow all logs
journalctl -f

# Follow specific service logs
journalctl -fu nginx.service
```

### **Debugging with -xe**
When a service fails, administrators often use `-xe`:
- `-x` (**catalog**): Adds explanatory help text for errors.
- `-e` (**pager-end**): Immediately jumps to the end of the journal.
```bash
journalctl -xe
```

### **Time-based Filtering**
`journalctl` supports very flexible time filters:
```bash
# Since a specific date/time
journalctl --since "2026-01-10 12:00:00"

# Using relative time
journalctl --since "1 hour ago"
journalctl --since yesterday
journalctl --since "2 days ago" --until "1 hour ago"
```

### **Filtering by Priority**
Filter logs by their severity levels (0: emerg to 7: debug):
```bash
# View errors, critical, alerts, and emergency messages
journalctl -p err -b
```

### **Persistent Logging**
By default, some distributions store logs in `/run/log/journal` (volatile, lost on reboot). To make logs persistent:
1. Ensure the directory exists:
   ```bash
   sudo mkdir -p /var/log/journal
   sudo systemd-tmpfiles --create --prefix /var/log/journal
   ```
2. Configure `/etc/systemd/journald.conf`:
   ```ini
   [Journal]
   Storage=persistent
   ```
3. Restart the service:
   ```bash
   sudo systemctl restart systemd-journald
   ```

## Interview Questions

**Q: How do you view the logs of a specific systemd service?**
**A:** Use the `-u` (unit) flag: `journalctl -u <service-name>`. You can also combine it with `-f` to follow the logs in real-time.

**Q: What is the advantage of using `journalctl -xe` when a service fails to start?**
**A:** The `-e` flag jumps to the end of the logs so you see the latest failure messages immediately. The `-x` flag provides "augmented" logs with metadata and links to documentation or common fixes from the systemd message catalog.

**Q: How do you view logs for the current boot only?**
**A:** Use the `-b` flag: `journalctl -b`. You can also view previous boots using offsets like `journalctl -b -1`.

**Q: How can you check how much disk space the journal logs are consuming?**
**A:** Use the `--disk-usage` flag: `journalctl --disk-usage`.

**Q: How do you filter logs to show only entries from the kernel?**
**A:** Use the `-k` or `--dmesg` flag: `journalctl -k`.

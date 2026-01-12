---
tags: ['linux', 'roadmap']
---

# Authentication Logs

## Summary
Authentication logs are critical for security auditing and troubleshooting user access in Linux systems. They track login attempts (successful and failed), session durations, and administrative actions performed via `sudo`. Depending on the distribution, these logs are primarily stored in `/var/log/auth.log` (Debian-based) or `/var/log/secure` (Red Hat-based). Tools like `last`, `lastb`, and `lastlog` provide structured ways to query binary log files like `wtmp`, `btmp`, and `lastlog`.

## Detailed Explanation

### 1. Primary Log Files
The location of authentication logs varies by Linux distribution:
- **Debian/Ubuntu**: `/var/log/auth.log`
- **RHEL/CentOS/Fedora**: `/var/log/secure`

These files record:
- SSH logins and logouts.
- `sudo` command executions.
- System service authentication (PAM - Pluggable Authentication Modules).
- Password changes and account lockouts.

### 2. Monitoring Failed Logins
Monitoring failed attempts is essential for detecting brute-force attacks.

**Using `lastb`**:
The `lastb` command reads from `/var/log/btmp` (which stores bad login attempts) and lists them.
```bash
# Show all failed login attempts
sudo lastb

# Show the top 10 IP addresses with failed attempts
sudo lastb | awk '{print $3}' | sort | uniq -c | sort -nr | head -n 10
```

**Using `grep` on auth logs**:
```bash
# Find failed SSH attempts in Ubuntu
grep "Failed password" /var/log/auth.log

# Find failed SSH attempts in RHEL
grep "Failed password" /var/log/secure
```

### 3. Monitoring Successful Logins
**Using `last`**:
The `last` command reads from `/var/log/wtmp` and shows a history of successful logins and system reboots.
```bash
# Show last 10 successful logins
last -n 10

# Show logins for a specific user
last username
```

**Using `lastlog`**:
Displays the most recent login time for every user on the system by reading `/var/log/lastlog`.
```bash
# Show last login for all users
lastlog

# Show users who have never logged in
lastlog -b 0
```

### 4. Tracking Sudo Usage
When a user executes a command with `sudo`, it is logged with the user's name, the working directory, and the command executed.

```bash
# Search for sudo executions
grep "sudo" /var/log/auth.log | grep "COMMAND"

# Example output line:
# Jan 10 12:00:00 server sudo:  user1 : TTY=pts/0 ; PWD=/home/user1 ; USER=root ; COMMAND=/usr/bin/apt update
```

### 5. Systemd Journal
On modern systems, you can also query authentication logs using `journalctl`:
```bash
# View SSH authentication logs via journalctl
journalctl _COMM=sshd

# View sudo-specific logs
journalctl _COMM=sudo
```

## Interview Questions

**Q: What is the difference between `/var/log/auth.log` and `/var/log/secure`?**
**A:** They serve the same purpose (logging authentication events) but differ by distribution. `/var/log/auth.log` is used by Debian-based systems (Ubuntu, Debian), while `/var/log/secure` is used by Red Hat-based systems (RHEL, CentOS, Fedora).

**Q: Which binary files do the `last` and `lastb` commands read from?**
**A:** `last` reads from `/var/log/wtmp` (successful logins), and `lastb` reads from `/var/log/btmp` (failed login attempts).

**Q: How can you find the last 5 users who logged into the system?**
**A:** You can use the command `last -n 5`.

**Q: How would you identify a brute-force attack on SSH using log files?**
**A:** By checking for a high volume of "Failed password" entries from the same IP address in `/var/log/auth.log` or `/var/log/secure`, or by using `lastb` to see multiple failed attempts in a short period.

**Q: Where can you see which commands a user has run with `sudo`?**
**A:** `sudo` commands are logged in the main authentication log file (`/var/log/auth.log` or `/var/log/secure`). You can filter them using `grep "sudo" /var/log/auth.log`.

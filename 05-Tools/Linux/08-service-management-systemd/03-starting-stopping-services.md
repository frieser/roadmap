#Linux
---
tags: ['linux', 'roadmap', 'systemd']
---

## Summary
Managing services is a core task for Linux administrators. In modern Linux distributions, **systemd** uses the `systemctl` command to control the state of services (daemons). This includes starting, stopping, restarting, and configuring services to persist across reboots.

## Detailed Explanation

The `systemctl` utility is the central tool for controlling the systemd system and service manager. It replaces older Init scripts (`/etc/init.d/`) and the `service` command.

### Basic Service Control

- **Starting a Service**:
  Activates the service immediately.
  ```bash
  sudo systemctl start nginx
  ```

- **Stopping a Service**:
  Deactivates the service immediately.
  ```bash
  sudo systemctl stop nginx
  ```

- **Restarting a Service**:
  Stops and then starts the service. This results in a new Process ID (PID).
  ```bash
  sudo systemctl restart nginx
  ```

- **Reloading a Service**:
  Asks the service to reload its configuration files without stopping. The PID remains the same. Not all services support this.
  ```bash
  sudo systemctl reload nginx
  ```

### Managing Boot Persistence

Starting a service does **not** mean it will start automatically after a reboot. You must manage its "enabled" state.

- **Enable**: Creates a symbolic link in the systemd configuration directories (typically `/etc/systemd/system/multi-user.target.wants/`), ensuring the service starts at boot.
  ```bash
  sudo systemctl enable nginx
  ```

- **Disable**: Removes the symbolic link, preventing the service from starting automatically.
  ```bash
  sudo systemctl disable nginx
  ```

### Difference: Restart vs. Reload

| Feature | `restart` | `reload` |
| :--- | :--- | :--- |
| **Action** | Full stop and start | Re-reads configuration |
| **PID** | Changes | Remains the same |
| **Availability** | Always available | Must be supported by the service |
| **Downtime** | Brief interruption | No downtime |

### Advanced Management: Masking

If you want to ensure a service is **never** started (even by another service or manually), you can "mask" it. This links the service file to `/dev/null`.

- **Mask**: `sudo systemctl mask nginx`
- **Unmask**: `sudo systemctl unmask nginx`

## Interview Questions

### 1. What is the difference between `systemctl restart` and `systemctl reload`?
`restart` stops the service and starts it again, assigning a new PID and causing a brief period of downtime. `reload` sends a signal to the service to re-read its configuration files while staying running, maintaining the same PID and avoiding downtime.

### 2. Does `systemctl start service` ensure the service runs after a reboot?
No. `start` only affects the current session. To ensure it starts after a reboot, you must use `systemctl enable service`.

### 3. How do you check if a service is enabled to start at boot without looking at its full status?
You can use the `is-enabled` subcommand:
```bash
systemctl is-enabled nginx
```

### 4. What happens when you `mask` a service?
Masking a service makes it impossible to start, either manually or as a dependency, by linking its unit file to `/dev/null`. This is stronger than `disable`.

### 5. How can you start a service and enable it at the same time?
Use the `--now` flag with the `enable` command:
```bash
sudo systemctl enable --now nginx
```

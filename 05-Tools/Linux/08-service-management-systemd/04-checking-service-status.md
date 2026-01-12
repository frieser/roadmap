#Linux
---
tags: ['linux', 'roadmap']
---

## Summary
Checking the status of a service is a fundamental task in Linux system administration using `systemd`. The `systemctl` command provides various subcommands like `status`, `is-active`, and `is-enabled` to inspect the current state, boot-time configuration, and runtime health of system units. Understanding the detailed output of these commands and their exit codes is crucial for both manual troubleshooting and automated scripting.

## Detailed Explanation

The `systemctl` utility is the primary tool for introspecting the state of `systemd` services.

### 1. Detailed Status: `systemctl status`
The most common command for a human operator is `systemctl status <service>`. It provides a comprehensive overview:

```bash
systemctl status nginx
```

**Key components of the output:**
- **Loaded**: Shows if the unit file is loaded in memory, the absolute path to the file, and its enablement state (`enabled`, `disabled`, `static`, `masked`).
- **Active**: The current runtime state. 
    - `active (running)`: Service is up and running.
    - `active (exited)`: Common for one-shot services that completed successfully.
    - `inactive (dead)`: Service is not running.
    - `failed`: Service crashed or exited with an error.
- **Main PID**: The process ID of the main service daemon.
- **Tasks**: Number of processes/threads in the service's control group.
- **Memory**: Current memory consumption.
- **CGroup**: The Control Group hierarchy, showing all processes associated with the service.
- **Journal Logs**: The last few lines of log output from `journald`.

### 2. Scripting-friendly Checks: `is-active` and `is-enabled`
While `status` is great for humans, it is hard to parse in scripts. `systemd` provides specialized subcommands that return simple strings and specific exit codes.

#### `systemctl is-active`
Checks if the service is currently running.
- **Output**: `active`, `inactive`, `failed`, `activating`, `deactivating`.
- **Exit Code**: `0` if active, non-zero otherwise.

```bash
if systemctl is-active --quiet nginx; then
    echo "Nginx is running"
else
    echo "Nginx is down"
fi
```

#### `systemctl is-enabled`
Checks if the service is configured to start at boot.
- **Output**: `enabled`, `disabled`, `static`, `masked`, `alias`, `indirect`.
- **Exit Code**: `0` if enabled, non-zero otherwise.

```bash
systemctl is-enabled sshd
# Output: enabled
# Exit code: 0
```

### 3. Exit Codes Table
| Command | State | Exit Code |
|---------|-------|-----------|
| `is-active` | active | 0 |
| `is-active` | inactive / failed | non-zero |
| `is-enabled` | enabled | 0 |
| `is-enabled` | disabled / masked | non-zero |

### 4. Advanced Status Filtering
You can list the status of multiple units or filter them by state:
```bash
# List all failed services
systemctl --failed

# List all active services
systemctl list-units --type=service --state=active
```

## Interview Questions

**Q: What is the difference between an `enabled` service and an `active` service?**
**A:** An **enabled** service is configured to start automatically at boot time (it has a symlink in a `.target.wants` directory). An **active** service is one that is currently running in memory. A service can be enabled but not active (stopped manually) or active but not enabled (started manually but won't persist after reboot).

**Q: How do you check if a service is running using only the exit code in a Bash script?**
**A:** Use `systemctl is-active --quiet <service>`. The `--quiet` flag suppresses the string output, and the command returns `0` if active and a non-zero value if not.

**Q: What does it mean when a service status is `masked`?**
**A:** A **masked** service is a stronger version of "disabled". The unit file is symlinked to `/dev/null`, making it impossible to start the service (manually or as a dependency) until it is explicitly unmasked using `systemctl unmask`.

**Q: How can you see the specific reason why a service failed to start from the `systemctl status` output?**
**A:** The `Active: failed` line often includes an exit code (e.g., `code=exited, status=1/FAILURE`). Additionally, the bottom of the output contains the most recent logs from `journald`, which typically describe the error that caused the failure.

---
---

## Summary
Systemd Timers are the modern, powerful alternative to Cron. A timer is a `.timer` unit file that controls a `.service` unit file. They offer superior logging, second-level precision, dependency management, and random delays (jitter).

## Detailed Explanation

### Components
1.  **Service Unit (`mytask.service`)**: Defines *what* to run.
2.  **Timer Unit (`mytask.timer`)**: Defines *when* to run it.

### Timer Types
*   **Monotonic**: Relative to an event (boot, activation).
    *   `OnBootSec=15min` (Run 15m after boot).
    *   `OnUnitActiveSec=1h` (Run 1h after the service last ran).
*   **Realtime**: Calendar-based (like Cron).
    *   `OnCalendar=*-*-* 00:00:00` (Daily at midnight).

### Management
*   `systemctl start mytask.timer`
*   `systemctl list-timers` (Shows next run, last run).

## Go-Specific Context/Examples

When deploying a Go backend service, you often use systemd. Adding a timer is cleaner than adding a cron job because everything stays in the systemd ecosystem.

### Example: Go Cleanup Service
**cleanup.service**
```ini
[Service]
ExecStart=/usr/local/bin/my-go-app cleanup
```
**cleanup.timer**
```ini
[Timer]
OnCalendar=daily
Persistent=true  # Catch up if missed while off

[Install]
WantedBy=timers.target
```

## Interview Questions

**Q: Why use Systemd Timers over Cron?**
**A:**
1.  **Logging**: Stdout/Stderr goes to journald automatically.
2.  **Dependencies**: Wait for network/DB before running.
3.  **Jitter**: `RandomizedDelaySec` prevents thundering herds.
4.  **Monitoring**: Easy to see when it last ran and its exit code.

**Q: What does `Persistent=true` do?**
**A:** It mimics `anacron`. If the system was off during the scheduled time, the timer triggers immediately upon booting up.

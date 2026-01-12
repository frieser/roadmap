---
---

## Summary
`cron` is a time-based job scheduler in Unix-like operating systems. Users allow `cron` to schedule jobs (commands or scripts) to run periodically at fixed times, dates, or intervals. The configuration file is called a `crontab` (cron table).

## Detailed Explanation

### Syntax
`* * * * * command_to_execute`
1.  **Minute** (0-59)
2.  **Hour** (0-23)
3.  **Day of Month** (1-31)
4.  **Month** (1-12)
5.  **Day of Week** (0-7, 0/7=Sunday)

### Examples
*   `0 0 * * * /backup.sh`: Run daily at midnight.
*   `*/5 * * * * /ping.sh`: Run every 5 minutes.
*   `0 9-17 * * 1-5 /work.sh`: Run hourly 9am-5pm on weekdays.

### Management
*   `crontab -e`: Edit current user's cron.
*   `crontab -l`: List tasks.
*   `/etc/crontab`: System-wide cron (has an extra "user" column).

## Go-Specific Context/Examples

Standard cron only has 1-minute resolution. For Go apps needing second-level precision or internal scheduling, use a library like `robfig/cron`.

### Example: Internal Cron in Go
```go
package main

import (
	"fmt"
	"github.com/robfig/cron/v3"
)

func main() {
	c := cron.New()
	// Run every 10 seconds (extended syntax)
	c.AddFunc("@every 10s", func() { 
		fmt.Println("Tick every 10s") 
	})
	c.Start()
	select {} // Block forever
}
```

## Interview Questions

**Q: Where does the output of a cron job go?**
**A:** By default, cron emails the stdout/stderr to the user. If no mailer is configured, output is discarded. Best practice is to redirect explicitly: `* * * * * cmd >> /var/log/cron.log 2>&1`.

**Q: What environment variables are available in cron?**
**A:** Very few. Cron runs with a minimal shell environment (`PATH` is usually just `/usr/bin:/bin`). It does **not** load your `.bashrc` or `.profile`. You must explicitly set `PATH` or source your profile inside the script.

**Q: How do you prevent a cron job from overlapping (running a second instance if the first is still slow)?**
**A:** Use **locking**, e.g., `flock`.
`* * * * * /usr/bin/flock -n /tmp/myjob.lock /path/to/job.sh`.

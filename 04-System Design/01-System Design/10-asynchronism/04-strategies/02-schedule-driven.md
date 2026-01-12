---
---

## Summary
**Schedule-Driven** strategies involve executing tasks at specific times or intervals (Cron jobs). This is useful for batch processing, reporting, and maintenance tasks that do not need to be real-time.

## Detailed Explanation

### Cron Jobs
*   **Standard**: Unix cron format (`* * * * *`). Runs scripts at set times.
*   **Distributed Scheduler**: In a cluster, you don't want every server running the "Daily Email" job 10 times. You need a centralized scheduler (like Kubernetes CronJob or a leader-elected process) to ensure the job runs exactly once.

### Use Cases
*   **Reporting**: Generating daily PDF reports at 2 AM.
*   **Cleanup**: Deleting soft-deleted rows older than 30 days.
*   **Billing**: Charging monthly subscriptions.

## Go Context: Periodic Tasks
Using standard `time.Ticker` or libraries like `robfig/cron`.

```go
// Simple Ticker
ticker := time.NewTicker(24 * time.Hour)
for range ticker.C {
    RunDailyCleanup()
}

// Distributed Locking (Redis)
// Only the instance that acquires the lock runs the job
if redis.SetNX("lock:daily-cleanup", 1, 10*time.Minute) {
    RunDailyCleanup()
}
```

## Interview Questions

### Q: How do you prevent a Cron Job from overlapping?
**A:** If a job runs every minute but takes 90 seconds to complete, you'll have two instances running simultaneously. To prevent this, use a **lock file** or a distributed lock (Redis/Zookeeper). If the lock exists, the new run skips execution.

### Q: What happens if a Scheduled Job fails?
**A:** It depends on the design. Standard Cron just waits for the next interval. A robust Distributed Scheduler (like Airflow or Temporal) supports **Retries**, **Alerting**, and **Backfills** (running missed past jobs).

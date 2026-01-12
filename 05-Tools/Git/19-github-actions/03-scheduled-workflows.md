# Scheduled Workflows

## Summary
You can run workflows on a schedule using POSIX cron syntax. This is useful for nightly builds, dependency updates, or stale issue cleanup.

## Detailed Explanation

### Syntax
```yaml
on:
  schedule:
    # Runs at 00:00 UTC every day
    - cron: '0 0 * * *'
```

### Limitations
*   The minimum interval is 5 minutes.
*   Times are in UTC.
*   GitHub makes no guarantee of exact execution time (can be delayed during high load).

### Use Cases
*   **Nightly Builds**: Run expensive integration tests that take too long for PRs.
*   **Security Scans**: Periodically scan dependencies for vulnerabilities even if no code changed.

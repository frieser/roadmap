## Summary
Scheduled workflows use POSIX cron syntax to run at specific times. This is particularly useful for AI Engineers for periodic tasks like daily model evaluation, weekly data drift checks, or automated backup of training logs.

## Detailed Explanation
### **Cron Syntax**
The `schedule` trigger uses five fields: `minute hour day(month) month day(week)`.
```yaml
on:
  schedule:
    - cron: '0 0 * * *' # Every day at midnight UTC
```

### **AI Engineering Use Cases**
1. **Daily Evaluation**: Run the model against a "golden dataset" every night to ensure no performance degradation.
2. **Weekly Retraining**: Automatically fine-tune a model on the last week of collected data.
3. **Cache Cleanup**: Periodically clear old model checkpoints or temporary datasets to save storage.

### **Limitations**
- Scheduled workflows run on the `main` or `master` branch.
- The shortest interval is roughly 5 minutes, but GitHub may delay runs based on system load.

## Interview Questions
- **Q: What is the syntax for running a workflow every Sunday at 3 AM?**
- **A:** `cron: '0 3 * * 0'`.

- **Q: On which branch do scheduled workflows execute?**
- **A:** They execute using the workflow file from the default branch (usually `main`).

- **Q: Can you rely on scheduled workflows for precise, time-critical tasks?**
- **A:** No, because GitHub Actions may delay scheduled runs depending on high demand or system resources.

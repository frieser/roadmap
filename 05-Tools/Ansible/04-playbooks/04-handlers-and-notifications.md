---
---

## Summary
Handlers are special tasks that only run when **triggered** by another task. They are typically used for service restarts. If a task (like updating a config file) reports a "change", it notifies the handler. Handlers run once at the very end of the play, regardless of how many times they were notified.

## Detailed Explanation

### Workflow
1.  **Task**: Changes a config file (`changed=true`).
2.  **Notify**: The task has a `notify: Restart Nginx` line.
3.  **Queue**: Ansible flags the "Restart Nginx" handler to run.
4.  **End of Play**: After all tasks finish, triggered handlers execute.

### Example
```yaml
tasks:
  - name: Update Nginx Config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx  # Only triggers if file changed

handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

## Go-Specific Context/Examples

This pattern is analogous to **Event-Driven Architecture** in Go using channels.

### Analogy: Go Channels
A task is like a producer sending a signal to a buffered channel. The handler is a consumer that drains the channel at the end. Even if 5 signals are sent, the consumer typically performs the action once (debouncing) or processes the batch.

```go
// Conceptual Go Analogy
func task() {
    if configChanged {
        notifyChan <- "restart_service"
    }
}

func main() {
    runTasks()
    
    // Handlers run at the end
    close(notifyChan)
    for event := range notifyChan {
        if event == "restart_service" {
            restartService()
            break // Run once
        }
    }
}
```

## Interview Questions

**Q: If a task notifies a handler but the play fails before reaching the end, does the handler run?**
**A:** By default, **No**. Handlers run at the end of the play. If the playbook crashes midway, handlers for previous successful tasks are skipped. You can force them to run using the `--force-handlers` command-line flag.

**Q: If a task notifies a handler 5 times, how many times does the handler run?**
**A:** **Once**. Ansible de-duplicates notifications. This is efficient; you don't want to restart Nginx 5 times just because you updated 5 different config files.

**Q: Can a handler notify another handler?**
**A:** Yes. This allows for chains of events (e.g., "Restart Apache" -> notify "Check Health").

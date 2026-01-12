---
---

## Summary
Ansible Blocks allow you to group multiple tasks together and apply common settings (like `when`, `become`, or `tags`) to the whole group. More importantly, they enable **Error Handling** through `rescue` (catch) and `always` (defer) sections, similar to `try-catch-finally` in programming.

## Detailed Explanation

### Structure
```yaml
tasks:
  - name: Attempt to upgrade database
    block:
      - name: Stop App
        service: name=app state=stopped
      - name: Run Upgrade Script
        command: /bin/upgrade_db.sh
    
    rescue:
      - name: Revert Database (Runs if block fails)
        command: /bin/restore_db.sh
      - name: Debug Failure
        debug: msg="Upgrade failed, rolling back..."

    always:
      - name: Restart App (Runs no matter what)
        service: name=app state=started
```

### Use Cases
1.  **Grouping**: Applying `when: ansible_os_family == "RedHat"` to 10 tasks at once.
2.  **Recovery**: Automatically rolling back changes if a deployment step fails.

## Go-Specific Context/Examples

This maps directly to Go's error handling and cleanup patterns.

### Analogy
**Ansible Block/Rescue/Always**:
```yaml
block: ...
rescue: ...
always: ...
```
**Go**:
```go
func main() {
    // Always (defer)
    defer restartApp()
    
    // Block
    if err := stopApp(); err != nil {
        handleError(err) // Rescue
        return
    }
    
    if err := upgradeDB(); err != nil {
        restoreDB() // Rescue
        return
    }
}
```

## Interview Questions

**Q: Does `rescue` mark the play as failed?**
**A:** No. If the `rescue` block succeeds, Ansible considers the entire task block as successful and continues the playbook. It "clears" the error state.

**Q: Can blocks be nested?**
**A:** Yes, you can put a block inside a block, though it can make the YAML hard to read.

**Q: Why use `block` instead of `ignore_errors`?**
**A:** `ignore_errors` just skips the failure and moves on. `block/rescue` allows you to take **specific corrective action** (rollback, notification) when a failure occurs.

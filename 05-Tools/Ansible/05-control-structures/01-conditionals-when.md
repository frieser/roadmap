---
---

## Summary
The `when` statement is Ansible's conditional mechanism. It allows a task to execute only if a specific condition is true, such as checking the OS version, verifying if a file exists, or checking the result of a previous task.

## Detailed Explanation

### Syntax
```yaml
tasks:
  - name: Install Apache
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "Debian"

  - name: Install HTTPD
    yum:
      name: httpd
      state: present
    when: ansible_os_family == "RedHat"
```

### Advanced Conditions
*   **Multiple conditions (AND)**:
    ```yaml
    when: 
      - ansible_distribution == "Ubuntu"
      - ansible_distribution_major_version == "20.04"
    ```
*   **OR condition**: `when: ansible_os_family == "Debian" or ansible_os_family == "RedHat"`
*   **Checking Registered Variables**:
    ```yaml
    - command: /bin/check_app
      register: result
      ignore_errors: yes

    - name: Restart App
      service: name=app state=restarted
      when: result.rc == 0
    ```

## Go-Specific Context/Examples

This is equivalent to `if` statements in Go.

### Analogy
**Ansible**:
```yaml
when: debug_mode | bool
```
**Go**:
```go
if config.DebugMode {
    // Run task
}
```

## Interview Questions

**Q: What is the difference between `when` and `failed_when`?**
**A:** `when` determines **if** the task should run at all. `failed_when` determines **if** the task result should be considered a failure (e.g., fail only if stderr contains "CRITICAL").

**Q: Can you use Jinja2 templating `{{ }}` inside a `when` clause?**
**A:** **No**. The `when` clause is already an implicit Jinja2 expression context. You should write `when: var == "value"`, NOT `when: {{ var }} == "value"`.

**Q: How do you check if a variable is defined?**
**A:** `when: my_var is defined`.

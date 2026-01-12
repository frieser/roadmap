---
---

## Summary
Ansible roles enforce a standard directory structure. Each directory contains a `main.yml` file which acts as the entry point for that component. You don't need all directories, only the ones you use.

## Detailed Explanation

### Standard Layout
```text
roles/
  my_role/
    tasks/       # Main list of tasks to execute
    handlers/    # Handlers (services to restart)
    defaults/    # Default variables (lowest priority)
    vars/        # Role variables (higher priority)
    files/       # Static files to copy (certificates, binaries)
    templates/   # Jinja2 templates (.j2)
    meta/        # Metadata (dependencies, author info)
    tests/       # Test inventory and playbook
```

### Purpose of Key Directories
*   **`defaults/main.yml`**: Vars intended to be overwritten by the user.
*   **`vars/main.yml`**: Vars internal to the role (should not be overwritten).
*   **`files/`**: Used by `copy` module. No path needed (`src: myfile.txt` looks here automatically).
*   **`templates/`**: Used by `template` module.

## Go-Specific Context/Examples

This mimics the **Standard Go Project Layout** (`cmd/`, `internal/`, `pkg/`, `api/`). It provides a common language for developers so they know exactly where to look for logic vs config.

*   `tasks/` ≈ `pkg/service/logic.go`
*   `defaults/` ≈ `config/default.yaml`
*   `tests/` ≈ `_test.go`

## Interview Questions

**Q: If a variable is defined in both `defaults/main.yml` and `vars/main.yml`, which one wins?**
**A:** **`vars/main.yml`** wins. `defaults` has the lowest precedence in Ansible—it is meant to provide "safe fallbacks" that users can easily override in their inventory or playbook variables. `vars` is meant for constants or internal logic.

**Q: How do you create the skeleton of a new role automatically?**
**A:** Run `ansible-galaxy init my_role_name`. It creates the directory tree for you.

**Q: What goes in `meta/main.yml`?**
**A:** Role dependencies (other roles that must run first) and Galaxy metadata (author, license, supported platforms).

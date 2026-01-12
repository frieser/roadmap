---
---

## Summary
Roles are the primary mechanism for breaking a playbook into multiple files. They simplify writing complex playbooks by grouping related tasks, variables, files, and handlers into a standard directory structure. A Role is essentially a reusable "package" of Ansible automation (e.g., a `mysql` role, a `webserver` role).

## Detailed Explanation

### Benefits
1.  **Modularity**: Break huge `site.yml` files into small, manageable components.
2.  **Reusability**: Write a role once, use it in multiple projects.
3.  **Sharing**: Share roles via Ansible Galaxy (the package registry).

### Using a Role
In your playbook:
```yaml
- hosts: webservers
  roles:
    - common
    - nginx
    - { role: app, app_port: 8080 } # Passing variables
```

## Go-Specific Context/Examples

Roles are to Ansible what **Packages (or Modules)** are to Go.

### Analogy
*   **Playbook (`site.yml`)** = `main.go` (The entry point).
*   **Role (`roles/nginx`)** = `github.com/my/project/pkg/nginx` (The library code).
*   **Ansible Galaxy** = `pkg.go.dev` (The registry).

Just as you wouldn't write your whole Go app in `main.go`, you shouldn't put all tasks in `site.yml`.

## Interview Questions

**Q: What is the difference between `import_role` and `include_role`?**
**A:**
*   **`import_role` (Static)**: Ansible parses the role *before* the play starts. It's like copying and pasting the tasks into the playbook. Tags work better here.
*   **`include_role` (Dynamic)**: Ansible parses the role *during* execution (runtime). Allows looping over a role or choosing a role based on a variable.

**Q: Where does Ansible look for roles?**
**A:**
1.  A `roles/` directory relative to the playbook.
2.  `/etc/ansible/roles`.
3.  Paths defined in `roles_path` in `ansible.cfg`.

**Q: Can a role depend on another role?**
**A:** Yes, defined in `meta/main.yml`. If Role A depends on Role B, Ansible ensures Role B runs before Role A automatically.

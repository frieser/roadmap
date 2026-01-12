---
---

## Summary
Creating custom roles allows you to package your own automation logic into reusable, modular components. A well-designed role should be focused (do one thing well), configurable via variables, and idempotent.

## Detailed Explanation

### Creating the Skeleton
Use the CLI to generate the standard directory structure:
```bash
ansible-galaxy init my_custom_role
```

### Developing the Role
1.  **`tasks/main.yml`**: Add your modules. Use variables (`{{ package_name }}`) instead of hardcoded strings.
2.  **`vars/main.yml`**: Define constants (OS-specific paths).
3.  **`defaults/main.yml`**: Define default values for variables users might want to override.
4.  **`handlers/main.yml`**: Define service restarts.

### Best Practices
*   **Namespace Variables**: Prefix variables with the role name to avoid collisions (e.g., `nginx_port` instead of just `port`).
*   **Test It**: Use Molecule (or just a test playbook) to verify the role works on a clean container.
*   **Idempotency**: Ensure running the role twice doesn't break anything.

## Go-Specific Context/Examples

Think of a Custom Role as a **Go Library**.

*   **Public API**: The variables in `defaults/main.yml`. These are the "Exported" fields users can configure.
*   **Private Logic**: The `tasks/main.yml`. Users shouldn't need to touch this.
*   **Documentation**: The `README.md` is critical, just like `godoc`.

### Example: A Go-App Deployment Role
Structure for a role that deploys a generic Go binary:
```yaml
# defaults/main.yml
go_app_name: "myapp"
go_app_binary_path: "/usr/local/bin/{{ go_app_name }}"
go_app_service_user: "www-data"

# tasks/main.yml
- name: Copy Binary
  copy:
    src: "{{ go_app_name }}"
    dest: "{{ go_app_binary_path }}"
    mode: '0755'
  notify: Restart App

# handlers/main.yml
- name: Restart App
  service:
    name: "{{ go_app_name }}"
    state: restarted
```

## Interview Questions

**Q: How do you include OS-specific variables in a role?**
**A:** A common pattern is to use `include_vars` based on `ansible_os_family`.
```yaml
- include_vars: "{{ ansible_os_family }}.yml"
```
This loads `Debian.yml` or `RedHat.yml` dynamically, allowing you to handle differences like `apache2` vs `httpd` transparently.

**Q: What is the `meta/main.yml` file for?**
**A:** It contains metadata about the role (author, license) and strictly enforces **dependencies**. If your role needs `geerlingguy.docker` to run first, you list it in `dependencies: []` here.

**Q: Why prefix variables with the role name?**
**A:** Ansible has a flat global variable namespace. If two roles both use a variable named `port`, one will overwrite the other silently. Using `nginx_port` and `mysql_port` prevents this collision.

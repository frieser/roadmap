---
---

## Summary
Ansible Collections are a distribution format introduced to bundle roles, modules, plugins, and documentation into a single package. They solve the scalability issues of the old "flat" role structure and allow partners (AWS, Cisco, F5) to release updates independently of the main Ansible core release cycle.

## Detailed Explanation

### Structure
A collection is namespaced: `namespace.collection_name`.
*   **Namespace**: The vendor or author (e.g., `amazon`, `community`).
*   **Name**: The specific collection (e.g., `aws`, `general`).

Example: `amazon.aws.ec2_instance` (Module inside a collection).

### Using Collections
1.  **Install**: `ansible-galaxy collection install community.kubernetes`
2.  **Playbook**: Use the Fully Qualified Collection Name (FQCN).
    ```yaml
    - hosts: localhost
      tasks:
        - name: Create K8s Namespace
          community.kubernetes.k8s:
            kind: Namespace
            name: my-ns
    ```

### `collections` Keyword
You can simplify playbooks by declaring which collections to search:
```yaml
- hosts: all
  collections:
    - community.kubernetes
  tasks:
    - k8s: ... # No need for full prefix
```

## Go-Specific Context/Examples

Collections map directly to **Go Modules (v2+)**.

*   **Old Ansible**: Everything was in the "std lib" (Monolith).
*   **New Ansible**: Core is small (`ansible-core`), everything else is a module (`amazon.aws`).

This is similar to how Go moved `net/http` (std lib) vs `golang.org/x/net` (extended).

## Interview Questions

**Q: Why were Collections introduced?**
**A:** Before Collections, all modules lived in the main Ansible repository. This made the repo huge and meant you had to wait for a new Ansible release (every 6 months) to get a bug fix for a specific AWS module. Collections decouple the release cycles, allowing vendors to ship updates instantly.

**Q: What is an FQCN?**
**A:** Fully Qualified Collection Name. It is the unambiguous name of a module, e.g., `ansible.builtin.copy` vs just `copy`. Using FQCN is best practice to avoid ambiguity if two collections provide a module with the same name.

**Q: Where are collections installed?**
**A:** By default, `~/.ansible/collections`.

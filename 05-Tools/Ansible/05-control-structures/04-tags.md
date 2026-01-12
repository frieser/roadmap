---
---

## Summary
Tags allow you to label specific tasks or plays so you can run (or skip) only parts of a large playbook. This is essential for debugging or re-running specific steps (e.g., "only update configuration, don't reinstall packages") without executing the entire deployment.

## Detailed Explanation

### Assigning Tags
```yaml
tasks:
  - name: Install Packages
    apt: name=nginx
    tags: 
      - setup
      - install

  - name: Copy Config
    copy: src=nginx.conf dest=/etc/nginx/
    tags: 
      - config
```

### Running Tags
*   **Run only config**: `ansible-playbook site.yml --tags "config"`
*   **Skip setup**: `ansible-playbook site.yml --skip-tags "setup"`
*   **List tasks**: `ansible-playbook site.yml --list-tags`

### Special Tags
*   **`always`**: Tasks with this tag run even if you select other tags (unless explicitly skipped). Good for cleanup or checking connectivity.
*   **`never`**: Tasks with this tag only run if you explicitly request them. Good for dangerous debug tasks.

## Go-Specific Context/Examples

This is conceptually similar to **Go Build Tags**.

### Analogy
**Ansible**: `ansible-playbook --tags "integration"`
**Go**: `go test --tags=integration`

Both mechanisms allow you to selectively compile/execute subsets of code based on labels.

## Interview Questions

**Q: If I run with `--tags "config"`, will tasks without tags run?**
**A:** No. Only tasks explicitly tagged with "config" (or "always") will run. Untagged tasks are skipped.

**Q: Can I apply tags to a Block or a Role?**
**A:** Yes. If you apply a tag to a Block or a Role import, **all** tasks inside that block/role inherit that tag.
```yaml
- import_role:
    name: common
  tags: base
```

**Q: What is the `untagged` special tag?**
**A:** It allows you to run only the tasks that do *not* have any tags. `ansible-playbook site.yml --tags "untagged"`.

---
---

## Summary
Loops allow you to repeat a task for a list of items, such as installing multiple packages, creating multiple users, or changing permissions on multiple files. `loop` is the modern standard, replacing the older `with_items` (though `with_items` is still widely used).

## Detailed Explanation

### Syntax (`loop`)
```yaml
- name: Install Packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - vim
```

### Looping over Dictionaries
Used when you need multiple attributes per item (e.g., User + Group).
```yaml
- name: Create Users
  user:
    name: "{{ item.name }}"
    group: "{{ item.group }}"
  loop:
    - { name: 'alice', group: 'admin' }
    - { name: 'bob', group: 'dev' }
```

### `loop` vs `with_items`
*   **`with_items`**: Automatically flattens lists (if you pass a list of lists, it iterates over every sub-item).
*   **`loop`**: Does NOT flatten. It iterates exactly over what you give it. It is recommended for new playbooks.

## Go-Specific Context/Examples

This is exactly like `for range` loops in Go.

### Analogy
**Ansible**:
```yaml
loop:
  - a
  - b
```
**Go**:
```go
items := []string{"a", "b"}
for _, item := range items {
    install(item)
}
```

## Interview Questions

**Q: How do you access the current loop index?**
**A:** You can use `loop_control` with `index_var`.
```yaml
loop: [a, b, c]
loop_control:
  index_var: idx
# Access via {{ idx }}
```

**Q: Can you use `register` with a loop?**
**A:** Yes. The registered variable will contain a `results` key, which is a list of outputs for each iteration. `{{ my_var.results[0].stdout }}`.

**Q: Why use `package` module with a list instead of a loop?**
**A:** For package managers (`apt`, `yum`), passing the list directly to the `name` parameter (e.g., `name: [git, curl]`) is much faster than looping. A loop runs `apt-get install` once *per item*. Passing a list runs `apt-get install git curl` *once*.

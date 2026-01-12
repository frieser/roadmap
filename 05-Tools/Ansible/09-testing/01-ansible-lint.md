---
---

## Summary
Ansible Lint is a command-line tool that checks playbooks for bugs, syntax errors, and suspicious constructs. It enforces best practices (idempotency, readability, security) and helps standardize code style across a team. It goes beyond simple YAML syntax checking by understanding Ansible-specific logic.

## Detailed Explanation

### What it Checks
*   **Syntax**: Valid YAML.
*   **Idempotency**: Warns if a command is likely to change things every run (e.g., using `shell` without `creates`).
*   **Best Practices**:
    *   "Always use `become` instead of `sudo`".
    *   "Use FQCN (Fully Qualified Collection Names) for modules".
    *   "Permissions should be set when creating files".
*   **Deprecated**: Warns about removed features.

### Usage
```bash
# Install
pip install ansible-lint

# Run
ansible-lint site.yml
```

### Configuration
You can configure rules in `.ansible-lint` file.
```yaml
skip_list:
  - '204'  # Lines should be <= 160 chars
  - '301'  # Commands should not change things if nothing needs doing
```

## Go-Specific Context/Examples

You can integrate `ansible-lint` into a Go-based CI runner or pre-commit hook tool.

### Example: Running Lint from Go Test
If you are writing a Go tool that generates Ansible code, you should validate the output.

```go
package main

import (
	"fmt"
	"os/exec"
	"testing"
)

func TestGeneratedPlaybookLint(t *testing.T) {
	// Generate playbook...
	
	// Lint it
	cmd := exec.Command("ansible-lint", "generated_playbook.yml")
	output, err := cmd.CombinedOutput()
	
	if err != nil {
		t.Fatalf("Lint failed: %v\nOutput: %s", err, string(output))
	}
	fmt.Println("Lint passed!")
}
```

## Interview Questions

**Q: Why does `ansible-lint` complain about using `command` or `shell` modules?**
**A:** Because they are rarely idempotent by default. `shell: echo "hello" >> file.txt` will append "hello" every single time it runs. Lint warns you to use the `copy`, `template`, or `lineinfile` modules instead, which handle state correctly. If you *must* use shell, you should add `changed_when: false` or a `creates` condition.

**Q: How do you ignore a specific rule for just one task?**
**A:** Add a `noqa` comment or tag to the task.
```yaml
- name: Run dangerous command
  shell: rm -rf /tmp/*
  args:
    warn: false
  tags:
    - skip_ansible_lint  # Old way
# New way (comment):
# noqa: command-instead-of-module
```

**Q: Does passing `ansible-lint` guarantee the playbook will run?**
**A:** No. Linting is static analysis. It catches style/syntax errors. It cannot know if the SSH key is wrong, if the remote server is down, or if the package name "ngnix" is a typo (unless it checks a package DB, which it doesn't). You still need testing (Molecule).

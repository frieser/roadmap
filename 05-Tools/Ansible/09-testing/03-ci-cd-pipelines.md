---
---

## Summary
Integrating Ansible into CI/CD pipelines (GitLab CI, GitHub Actions, Jenkins) brings DevOps practices to infrastructure code. It ensures that changes to playbooks are automatically linted, tested, and deployed, enforcing quality and reducing manual errors.

## Detailed Explanation

### Typical Pipeline Stages
1.  **Lint**: Run `ansible-lint` and syntax checks. Fails on bad YAML.
2.  **Test**: Run `molecule test`. Fails if the role doesn't work or isn't idempotent.
3.  **Staging Deploy**: Run `ansible-playbook -i staging` against a dev environment.
4.  **Production Deploy**: (Manual Gate) Run `ansible-playbook -i production`.

### Handling Secrets
Never store secrets in the repo. Use the CI platform's secret manager (GitHub Secrets) to inject:
1.  **SSH Private Key**: For connecting to servers.
2.  **Ansible Vault Password**: For decrypting repo secrets.

## Go-Specific Context/Examples

If you write a Go app, your pipeline might look like this:
1.  **Go Build**: Compile binary.
2.  **Docker Build**: Package binary.
3.  **Ansible Deploy**: Trigger Ansible to pull the new image/binary and restart services.

### Example: GitHub Actions Workflow
```yaml
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install Ansible
        run: pip install ansible

      - name: Deploy to Staging
        run: ansible-playbook site.yml -i inventory/staging
        env:
          ANSIBLE_HOST_KEY_CHECKING: 'False'
          ANSIBLE_PRIVATE_KEY_FILE: key.pem
```

## Interview Questions

**Q: How do you handle SSH keys in a CI runner safely?**
**A:** Store the private key as a Protected Secret (e.g., `SSH_KEY`). In the pipeline job, start `ssh-agent`, add the key from the environment variable, or write it to a temporary file with restricted permissions (`chmod 600`) just for the duration of the job.

**Q: What is "GitOps" in the context of Ansible?**
**A:** GitOps means the Git repository is the "Source of Truth". You don't run Ansible manually from your laptop. You push a change to Git, and an automated agent (CI Pipeline or AWX Webhook) detects the change and applies the playbook to match the infrastructure to the Git state.

**Q: Why use a "Check Mode" run in CI?**
**A:** Running `ansible-playbook --check` in the pipeline allows you to see *what would happen* (Dry Run) without actually changing anything. This is useful for reviewing the impact of a Pull Request before merging.

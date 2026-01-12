---
tags: ['terraform', 'iac', 'tools', 'roadmap', 'github-actions', 'ci-cd']
---

# Terraform + GitHub Actions

## Summary
**GitHub Actions (GHA)** is the most popular CI/CD platform for Terraform due to its native integration with GitHub repositories and its powerful ecosystem of actions. The modern workflow leverages **OIDC (OpenID Connect)** for passwordless authentication and provides deep integration with Pull Requests for infrastructure review.

## Detailed Explanation: Automation Pipelines

### Core Workflow Pattern
The standard Terraform pipeline in GitHub Actions follows a multi-stage approach to ensure safety and visibility:

1.  **Setup & Validation**:
    *   **Checkout**: Uses `actions/checkout` to pull the code.
    *   **Setup Terraform**: Uses `hashicorp/setup-terraform` to install the CLI and configure wrappers for output capturing.
    *   **Linting**: Runs `terraform fmt -check` and tools like `tflint` or `tfsec`.
2.  **Initialization**: 
    *   Executes `terraform init` to download providers and initialize the remote backend.
3.  **The Plan Stage (Pull Request)**:
    *   Runs `terraform plan` on every PR.
    *   **PR Integration**: Uses `actions/github-script` to post the plan results as a comment on the PR, allowing reviewers to see the exact changes without leaving the UI.
4.  **The Apply Stage (Merge to Main)**:
    *   Executes `terraform apply` after the PR is merged.
    *   **Approval Gates**: Uses **GitHub Environments** to require manual approval before production deployment.

### Authentication with OIDC
Instead of storing long-lived `AWS_ACCESS_KEY_ID` secrets, GHA can use OIDC:
*   GitHub provides a short-lived JSON Web Token (JWT).
*   The cloud provider (AWS/GCP/Azure) trusts the GitHub OIDC provider.
*   The token is exchanged for temporary cloud credentials.
*   **Permissions required**: `id-token: write` and `contents: read`.

### State Management
*   **Remote Backend**: Always use a remote backend (S3/DynamoDB, GCS, or Terraform Cloud).
*   **State Locking**: Crucial for CI/CD to prevent concurrent modifications from different workflow runs.

## Go Context: Testing and Execution
In a Go-centric environment, GitHub Actions can be used to run **Terratest** or custom automation tools built with **terraform-exec**.

### Terratest in GHA
```yaml
- name: Run Terratest
  run: |
    cd test
    go test -v -timeout 30m
```

## Interview Questions

**Q: Why is OIDC preferred over GitHub Secrets for cloud provider authentication?**
**A:** OIDC eliminates the need for long-lived static credentials. It uses short-lived tokens and trust relationships, significantly reducing the security risk associated with credential leaks.

**Q: How do you implement a manual approval step in GitHub Actions for Terraform Apply?**
**A:** By using **GitHub Environments**. You define an environment (e.g., `production`) with "Required Reviewers" enabled, and reference that environment in the `apply` job within the workflow YAML.

**Q: What is the purpose of the `hashicorp/setup-terraform` action?**
**A:** It installs the Terraform CLI on the runner, adds it to the PATH, and optionally configures a wrapper that captures stdout/stderr, making it easier to parse and post plan results to PR comments.

**Q: How can you prevent two CI runs from modifying the same state simultaneously?**
**A:** By using a **remote backend with locking capabilities** (e.g., S3 with a DynamoDB table). Terraform will automatically attempt to acquire a lock before any operation that modifies the state.

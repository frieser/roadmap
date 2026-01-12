---
tags: ['terraform', 'iac', 'tools', 'roadmap', 'gitlab-ci', 'ci-cd']
---

# Terraform + GitLab CI

## Summary
**GitLab CI** offers a "batteries-included" experience for Terraform, featuring native integrations that set it apart. It provides **GitLab-managed Terraform State** (an internal HTTP backend), integrated Merge Request reports for plans, and a dedicated Terraform registry, making it a powerful all-in-one solution for IaC.

## Detailed Explanation: Automation Pipelines

### Native GitLab Integration
GitLab's Terraform integration is designed to be seamless, often requiring minimal configuration through the use of built-in templates.

1.  **GitLab-Managed State**:
    *   GitLab provides a built-in HTTP backend. You don't need to configure S3 or GCS for state files.
    *   State is encrypted at rest and tied to GitLab's authentication/authorization system.
2.  **Merge Request Widget**:
    *   By outputting a `terraform` report artifact, GitLab displays a summary of planned changes (Add/Change/Delete) directly in the Merge Request UI.
3.  **Pipeline Stages**:
    *   **Validate**: Runs `fmt` and `validate`.
    *   **Build**: Runs `terraform plan`. The JSON output is saved as a report.
    *   **Deploy**: Runs `terraform apply`. Typically uses `when: manual` for production.

### Environment-Scoped Variables
GitLab allows you to define CI/CD variables (like `AWS_ACCESS_KEY_ID`) and scope them to specific **Environments** (e.g., `production`, `staging`). This ensures that the production credentials are only injected into the production pipeline.

### Protected Environments
You can protect specific environments in GitLab so that only authorized users (e.g., maintainers) can trigger the deployment jobs.

## Go Context: Infrastructure Testing
GitLab CI can easily run **Terratest** by using a Docker image that includes both Go and Terraform.

### Example: `.gitlab-ci.yml` for Terratest
```yaml
test:
  image: hashicorp/terraform:latest
  script:
    - apk add go
    - go test -v ./test
```

## Interview Questions

**Q: How does GitLab display Terraform plan results in a Merge Request?**
**A:** By using the `artifacts:reports:terraform` keyword in the `.gitlab-ci.yml`. GitLab parses the plan's JSON output and renders a summary widget in the Merge Request, allowing reviewers to quickly see the impact of changes.

**Q: What are the advantages of using GitLab-managed Terraform state?**
**A:** It simplifies infrastructure by removing the need for external backends (like S3/DynamoDB). It provides built-in encryption, versioning, and access control that is natively integrated with GitLab's user management.

**Q: What is the purpose of the `TF_HTTP_ADDRESS` variable in GitLab CI?**
**A:** It is used to point Terraform to the GitLab-managed state backend. GitLab automatically populates this and other `TF_HTTP_*` variables when using the standard Terraform template.

**Q: How can you restrict who can run a `terraform apply` in GitLab?**
**A:** By using **Protected Environments**. You can specify that only users with "Maintainer" access or specific members can trigger jobs associated with a protected environment like `production`.

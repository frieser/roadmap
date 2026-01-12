---
tags: ['tools', 'roadmap']
---

## Summary
Terraform deployment workflows bridge the gap between local development and automated, high-scale infrastructure management. A robust workflow moves from a manual **Write → Plan → Apply** loop to an automated **CI/CD** or **GitOps** model, ensuring that changes are peer-reviewed, tested, and deployed securely using short-lived credentials (OIDC).

## Detailed Explanation

### 1. The Standard CLI Workflow
For individual development or small projects, the workflow is manual:
- **Write:** Authoring `.tf` files.
- **Plan (`terraform plan`):** Generating an execution plan to preview changes.
- **Apply (`terraform apply`):** Executing the plan to update infrastructure.
- **Destroy (`terraform destroy`):** Removing all managed resources.

**Best Practice:** Use `-out=tfplan` with `plan` and provide that file to `apply` to ensure you only apply exactly what you reviewed.

### 2. CI/CD Pipeline Workflow
In team environments, Terraform runs in a CI/CD system (e.g., GitHub Actions, GitLab CI):
- **Continuous Integration (CI):** Every Pull Request triggers a `terraform init`, `terraform fmt -check`, and `terraform plan`. The plan output is often posted as a PR comment.
- **Continuous Deployment (CD):** Merging to the main branch triggers `terraform apply`. For production, this usually requires a manual approval gate.

### 3. GitOps and Pull Request Automation
GitOps makes the Git repository the "Single Source of Truth."
- **Atlantis:** An open-source tool that runs as a server. It listens for PR comments like `atlantis plan` or `atlantis apply`. It locks the state to prevent conflicts between multiple PRs.
- **Terraform Cloud (TFC):** A managed service that provides remote state, remote execution, and VCS-driven workflows.

### 4. Security and Secrets Management
- **OIDC (OpenID Connect):** Modern CI/CD systems use OIDC to authenticate with cloud providers (AWS, Azure, GCP) without storing long-lived, static credentials (API keys).
- **Sensitive Variables:** Use the `sensitive = true` flag in HCL to redact values from logs and console output.

### GitHub Actions Example
A simplified workflow for planning on PRs.

```yaml
name: Terraform Plan
on: [pull_request]

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      
      - name: Terraform Init
        run: terraform init
        
      - name: Terraform Plan
        run: terraform plan -input=false
```

## Interview Questions

**Q: What are the benefits of running Terraform in a CI/CD pipeline versus locally?**
**A:** CI/CD provides a consistent environment, automated peer reviews (via plans on PRs), a clear audit trail of who deployed what, and better security by using short-lived credentials like OIDC.

**Q: What is Atlantis and how does it help with Terraform scaling?**
**A:** Atlantis is a GitOps tool that automates Terraform via Pull Request comments. It helps scale by allowing team members to perform plans and applies directly in the PR interface, handling state locking and providing visibility to the whole team.

**Q: How do you handle secrets (like database passwords) in a Terraform CI/CD pipeline?**
**A:** Secrets should never be hardcoded. Use environment variables (prefixed with `TF_VAR_`), fetch them from a Secret Manager (AWS Secrets Manager, Vault) during execution, or use OIDC to avoid storing long-lived cloud credentials entirely.

**Q: Why is it important to use `terraform plan -out=tfplan` in a pipeline?**
**A:** It ensures that the exact changes reviewed during the `plan` stage are the ones executed during the `apply` stage. Without it, the infrastructure state could change between the plan and the apply, leading to unexpected results.

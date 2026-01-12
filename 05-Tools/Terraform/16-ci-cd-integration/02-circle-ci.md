---
tags: ['terraform', 'iac', 'tools', 'roadmap', 'circle-ci', 'ci-cd']
---

# Terraform + CircleCI

## Summary
**CircleCI** is a robust CI/CD platform that excels in managing complex Terraform workflows through its modular **Orb** system. It provides powerful **Contexts** for managing environment-specific secrets and uses **Workspaces** to pass validated plan files between pipeline stages safely.

## Detailed Explanation: Automation Pipelines

### The "Orb" Advantage
CircleCI provides the official `circleci/terraform` orb, which encapsulates best practices into reusable jobs and commands. This reduces boilerplate and ensures consistency across different projects.

### Standard Pipeline Flow
1.  **Preparation**: 
    *   Uses the `terraform/install` command to set up a specific version.
    *   Runs `terraform fmt` and `terraform validate`.
2.  **Plan Generation**:
    *   Runs `terraform plan -out=tfplan`.
    *   **Persistence**: The `tfplan` file and the `.terraform` directory are persisted to the **CircleCI Workspace**. This is critical to ensure that the `apply` step uses the exact same plan that was generated.
3.  **Approval Gate**:
    *   Uses a `type: approval` job in the workflow. The pipeline pauses until a team member manually approves the deployment.
4.  **Application**:
    *   The `apply` job attaches the workspace, loads the `tfplan`, and executes `terraform apply tfplan`.

### Secret Management with Contexts
CircleCI **Contexts** allow you to group environment variables and share them across multiple projects. 
*   Example: A `terraform-prod` context contains AWS credentials for the production account.
*   Access can be restricted to specific users or branches.

### Environment Separation
Using different contexts for `staging` and `production` ensures that credentials for sensitive environments are only available to the jobs that strictly require them.

## Go Context: Programmatic Execution
For teams using Go, CircleCI can run integration tests using **Terratest** within specialized Docker executors.

### Example: Running Go tests
```yaml
jobs:
  test:
    docker:
      - image: cimg/go:1.21
    steps:
      - checkout
      - run: go test -v ./...
```

## Interview Questions

**Q: What is a CircleCI Orb and how does it help with Terraform?**
**A:** An Orb is a reusable package of YAML configuration. The Terraform Orb provides pre-defined jobs for init, plan, and apply, which handle common tasks like workspace persistence and command execution, following HashiCorp's best practices.

**Q: Why should you use `persist_to_workspace` for the Terraform plan file?**
**A:** It ensures that the `apply` job uses the **exact same plan** that was reviewed and approved in the `plan` job. This prevents the "race condition" where infrastructure changes occur between the plan and apply steps.

**Q: How do CircleCI Contexts improve security for Terraform pipelines?**
**A:** Contexts allow for the centralized management and encryption of secrets. They enable environment-based credential separation (e.g., separate keys for Dev vs. Prod) and can be restricted to specific authorized branches or users.

**Q: What is the benefit of using Docker-based executors for Terraform in CircleCI?**
**A:** It ensures a consistent, immutable environment for every run. By using official `hashicorp/terraform` or CircleCI-provided images, you avoid "it works on my machine" issues and ensure all required dependencies are present.

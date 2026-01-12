---
tags: ['tools', 'roadmap']
---

## Summary
Terragrunt is a Go-based thin wrapper for Terraform that provides extra tools for keeping configurations DRY (Don't Repeat Yourself), managing remote state, and orchestrating multiple Terraform modules. It is the industry standard for managing complex, multi-state Terraform deployments.

## Detailed Explanation

### Why Use Terragrunt?
While Terraform is powerful, it has limitations when scaling:
- **Redundant Backend Config**: Every module needs a `backend` block. Terragrunt can generate this automatically.
- **Redundant Provider Config**: Keeping provider versions in sync across many directories is difficult.
- **Manual Dependency Management**: Terraform doesn't natively handle dependencies between different state files easily.

### Core Features
1.  **DRY Backend/Provider Configuration**: You define your backend and provider configuration once in a root `terragrunt.hcl` file and inherit it in all child modules using `include`.
2.  **Dependency Orchestration**: Terragrunt allows you to define dependencies between modules.
    ```hcl
    dependency "vpc" {
      config_path = "../vpc"
    }
    inputs = {
      vpc_id = dependency.vpc.outputs.vpc_id
    }
    ```
3.  **Hooks**: Execute custom scripts or commands before or after Terraform commands (e.g., running `tflint` before `plan`).
4.  **CLI Pass-through**: Terragrunt passes all commands directly to Terraform, so you can run `terragrunt plan` just like `terraform plan`.

### Go Application
Terragrunt is written in **Go**. Its architecture leverages Go's concurrency for commands like `run-all`, which can apply multiple modules in parallel based on their dependency graph.

**Example Directory Structure with Terragrunt:**
```text
live/
├── terragrunt.hcl (Root config)
├── prod/
│   ├── vpc/
│   │   └── terragrunt.hcl
│   └── app/
│       └── terragrunt.hcl
└── staging/
    └── ...
```

### Terragrunt vs. CDKTF
- **Terragrunt**: Keeps you in the HCL ecosystem but adds a layer of orchestration and DRYness. Better for teams who prefer HCL.
- **CDKTF**: Moves you to a general-purpose language (like Go). Better for teams who want the power of a full programming language for infrastructure.

## Interview Questions

**Q: What is the primary purpose of Terragrunt?**
**A:** To keep Terraform configurations DRY and manage the orchestration of multiple Terraform modules/state files, which Terraform does not handle natively out of the box.

**Q: How does Terragrunt handle dependencies?**
**A:** It uses a `dependency` block to reference another module's state. When running `terragrunt run-all`, it builds a directed acyclic graph (DAG) to ensure modules are applied in the correct order.

**Q: Can you run Terraform commands through Terragrunt?**
**A:** Yes, Terragrunt acts as a wrapper. Any command you pass to `terragrunt` is forwarded to `terraform` after Terragrunt performs its own logic (like generating backend files).

**Q: What is the benefit of using Terragrunt for remote state management?**
**A:** It can automatically create the remote state bucket (e.g., S3) and locking table (e.g., DynamoDB) if they don't exist, and it allows you to define the backend configuration once for the entire project.

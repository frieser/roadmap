---
tags: ['tools', 'roadmap']
---

# Sentinel Policy as Code

## Summary
Sentinel is a **Policy as Code (PaC)** framework developed by HashiCorp to enforce fine-grained, logic-based policies across their enterprise suite (Terraform Cloud/Enterprise, Vault, Nomad). It sits between the execution plan and the application of changes, acting as a programmable guardrail that goes beyond simple static analysis to ensure compliance, security, and operational best practices.

## Detailed Explanation

Sentinel allows organizations to define complex policies that can evaluate Terraform plans before resources are provisioned. Unlike traditional ACLs which are often binary (allow/deny), Sentinel provides a Turing-complete language for expressing sophisticated logic.

### Core Concepts

1.  **Policy Hierarchy (Enforcement Levels)**:
    *   **Advisory**: Notifies the user of violations but allows the run to proceed.
    *   **Soft Mandatory**: Prevents the run if the policy fails, but can be overridden by an administrator.
    *   **Hard Mandatory**: Strictly prevents the run. No overrides are possible.
2.  **Imports**: Sentinel can import data sources to make decisions. In Terraform, common imports include `tfplan`, `tfstate`, `tfconfig`, and `tfrun`.
3.  **Rules**: The core logic of a policy, defined using logical expressions.

### Sentinel Syntax (HCL-like)

Sentinel policies use a language inspired by HCL but with full logical capabilities.

```hcl
import "tfplan/v2" as tfplan

# Define a list of allowed instance types
allowed_types = ["t3.micro", "t3.small", "t3.medium"]

# Rule to filter for aws_instance resources and check their instance_type
main = rule {
    all tfplan.resource_changes as _, rc {
        rc.type is not "aws_instance" or
        (rc.mode is "managed" and
         (rc.change.actions contains "create" or rc.change.actions contains "update") and
         rc.change.after.instance_type in allowed_types)
    }
}
```

### Why use Sentinel?
*   **Prevent Cost Overruns**: Limit expensive instance types or region choices.
*   **Security Compliance**: Ensure all S3 buckets are private or all EBS volumes are encrypted.
*   **Operational Guardrails**: Enforce mandatory tags or restrict deployments to specific time windows (e.g., no production changes on Fridays).
*   **External Integration**: Sentinel can query external APIs (e.g., check if a ticket is open in ServiceNow) before allowing a deployment.

### Go Integration Note
While Sentinel is a proprietary HashiCorp language, many Terraform practitioners use Go to write **Custom Providers** or **Terraform Plugins** that interact with the resources Sentinel governs. Testing Sentinel policies often involves the `sentinel-sdk`, which can be integrated into Go-based CI environments for policy validation.

## Interview Questions

**Q: What is the main difference between Sentinel and Terraform's built-in RBAC?**
**A:** RBAC (Role-Based Access Control) is identity-centric (Who can do what), while Sentinel is logic-centric (What conditions must be met). Sentinel allows for dynamic, conditional policies based on the content of the infrastructure plan itself.

**Q: Explain the "Soft Mandatory" enforcement level.**
**A:** Soft Mandatory allows a policy to block a run while still providing an "override" mechanism. This is useful for "exceptions to the rule" where a human administrator can manually approve a deployment that technically violates a policy.

**Q: How does Sentinel access Terraform data?**
**A:** Sentinel uses **Imports** like `tfplan` (the plan output), `tfstate` (current state), and `tfconfig` (the source code) to inspect the proposed changes and the current environment.

**Q: Can Sentinel policies prevent changes to existing resources?**
**A:** Yes, by importing `tfstate`, Sentinel can compare the proposed changes against the current state and block actions like the destruction of critical resources.

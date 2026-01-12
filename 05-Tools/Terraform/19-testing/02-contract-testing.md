---
tags: ['tools', 'roadmap']
---

## Summary
Contract testing in Terraform ensures that infrastructure modules adhere to a specific "contract"—a set of agreed-upon inputs, outputs, and behaviors—that other modules or applications depend on. Introduced in Terraform v1.5, **`check` blocks** provide a native way to perform non-blocking assertions to verify the state of infrastructure post-deployment, while the native test framework (v1.6+) allows for validating output contracts during the development phase.

## Detailed Explanation
### Core Concepts: `check` and `assert`
Terraform provides specific primitives for contract validation:
*   **`check` Blocks (v1.5+)**: These are **non-blocking** blocks that execute at the end of a `plan` or `apply`. If an assertion inside a `check` block fails, Terraform issues a warning but does not stop the execution. This is ideal for continuous validation or health monitoring.
*   **`assert` Blocks**: Used inside `check` blocks or `run` blocks in `.tftest.hcl`. They contain a `condition` and an `error_message`.

### Contract Testing vs. Other Types
Unlike unit testing (which tests logic) or integration testing (which tests creation), **contract testing** focuses on the **interface** and **assumptions** between systems.
*   **Input Contracts**: Validating that variables meet strict criteria.
*   **Output Contracts**: Ensuring a module provides the exact attributes expected by downstream consumers.
*   **State Contracts**: Verifying that the deployed infrastructure meets external standards (e.g., "The load balancer must return a 200 OK").

### Continuous Validation
In managed environments like HCP Terraform, `check` blocks can be used for **Continuous Validation**. They run periodically to ensure that the infrastructure hasn't drifted from its defined contract over time.

## HCL Code Examples
### Example 1: Post-Deployment Contract Validation (`check` block)
This example verifies that a web server is reachable and healthy after it has been deployed.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# Contract: The web server MUST be reachable via HTTP
check "health_check" {
  data "http" "web_status" {
    url = "http://${aws_instance.web.public_ip}/health"
  }

  assert {
    condition     = data.http.web_status.status_code == 200
    error_message = "The web server contract failed: /health endpoint is unreachable or returning error."
  }
}
```

### Example 2: Module Output Contract (`.tftest.hcl`)
This example verifies the "Output Contract" of a VPC module to ensure it provides the expected number of subnets.

```hcl
# tests/contract.tftest.hcl
variables {
  environment = "prod"
}

run "verify_vpc_outputs" {
  command = plan # Validates the output contract without creating resources

  assert {
    condition     = length(output.public_subnets) == 3
    error_message = "VPC module contract broken: Expected 3 public subnets for High Availability."
  }

  assert {
    condition     = can(regex("^vpc-", output.vpc_id))
    error_message = "VPC ID format does not match the expected contract."
  }
}
```

## Interview Questions
**Q: How do `check` blocks differ from `postcondition` blocks?**
**A:** `postcondition` blocks are **blocking**. If a `postcondition` fails, Terraform stops execution and prevents downstream resources from being created. `check` blocks are **non-blocking**; they run at the end of the operation and only issue warnings, making them suitable for health checks and non-critical validations.

**Q: What is a "Contract" in the context of Terraform modules?**
**A:** A contract is the agreed-upon interface between a module and its consumers. It includes the expected input variables (types, constraints), the provided output values (format, content), and the resulting state of the infrastructure (availability, security settings).

**Q: Why would you use `command = plan` in a contract test?**
**A:** It allows you to verify that the module **promises** the correct outputs and resource configurations. This validates the "interface contract" quickly and cheaply without actually deploying resources to the cloud.

**Q: Explain how to implement "Cross-Module Assertions".**
**A:** You can pass the output of one module into another as a variable. In the receiving module, you can use a `precondition` block on a resource or a `check` block to validate that the passed value meets the required contract before proceeding.

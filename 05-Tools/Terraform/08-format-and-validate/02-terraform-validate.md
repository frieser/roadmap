---
tags:
  - tools
  - roadmap
  - terraform
  - iac
---

## Summary
`terraform validate` is a static analysis tool that checks the syntax and internal consistency of Terraform configuration files in the current directory. It verifies that the code is syntactically valid (HCL compliance) and that attribute references match the provider's schema, all without accessing remote state or cloud APIs.

## Detailed Explanation

### What is `terraform validate`?
It is a command used to ensure your infrastructure code is correct before attempting to provision resources. It runs strictly locally and is often the first step in a CI/CD pipeline or a pre-commit hook.

Unlike `terraform plan`, which generates an execution plan by communicating with remote APIs and state files, `terraform validate` only looks at the code configuration files (`.tf`). It requires the directory to be initialized (`terraform init`) so that it has access to the provider plugins (schemas) against which it validates the resource attributes.

### How It Works

1.  **Syntax Check**: Verifies that the HCL (HashiCorp Configuration Language) structure is correct (e.g., matching braces, valid argument formats).
2.  **Schema Validation**: Checks if the defined resources and their attributes exist in the provider's schema and are of the correct type (e.g., checking if `instance_type` is a valid argument for `aws_instance`).
3.  **Internal Consistency**: Ensures that variable references and interpolations are syntactically correct and point to valid objects within the module.

### Usage

The basic command is run in the directory containing your Terraform files:

```bash
terraform validate
```

For automation scripts or CI pipelines, you might want machine-readable output:

```bash
terraform validate -json
```

To run it without color (good for logs):

```bash
terraform validate -no-color
```

### Examples

#### Success Output
When the configuration is valid:

```text
Success! The configuration is valid.
```

#### Failure Output
If there is a typo (e.g., `instanse_type` instead of `instance_type`):

```text
╷
│ Error: Unsupported attribute
│ 
│   on main.tf line 12, in resource "aws_instance" "web":
│   12:   instanse_type = "t3.micro"
│ 
│ This object has no attribute, named "instanse_type". Did you mean "instance_type"?
╵
```

### CI/CD Integration
In a pipeline (e.g., GitHub Actions), `terraform validate` serves as a fail-fast mechanism.

```yaml
# Example GitHub Actions step
- name: Terraform Validate
  run: terraform validate -no-color
```

## Interview Questions

**Q: What is the main difference between `terraform validate` and `terraform plan`?**
**A:** `terraform validate` is a static check that runs locally to verify syntax and internal consistency without connecting to remote services or checking the state file. `terraform plan` communicates with cloud providers and the state file to calculate the difference between the desired configuration and the real-world infrastructure.

**Q: Why must you run `terraform init` before `terraform validate`?**
**A:** `terraform validate` checks resource attributes against the provider's schema (rules). These schemas are part of the provider plugins (e.g., the AWS provider binary), which are downloaded to the `.terraform` directory during `terraform init`. Without them, Terraform doesn't know if a resource or argument is valid.

**Q: Does `terraform validate` detect if an AMI ID or S3 bucket name doesn't exist?**
**A:** No. It only validates correctness of the configuration (e.g., that the AMI ID is a string). It does not verify the existence or validity of the data values against the real cloud environment. That validation happens during `plan` or `apply`.

**Q: Can `terraform validate` check all subdirectories or modules at once?**
**A:** By default, `terraform validate` only checks the configuration in the current working directory. To validate nested modules, you would typically need to loop through directories or use a wrapper script/tool that handles recursive validation.

---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Output Syntax

## Summary

**Terraform outputs** return values from your configuration that can be used by other Terraform configurations or tools. Outputs make modules reusable, expose resource attributes, and facilitate communication between teams. Defined with `output` blocks, outputs can be sensitive (hidden from logs) and include descriptions for documentation.

## Detailed Explanation

### Output Syntax

```hcl
# Basic output
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

# Output with expression
output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}

# Multiple values
output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

# Sensitive output
output "database_password" {
  description = "Database connection password"
  value       = var.db_password
  sensitive   = true
}
```

### Output Arguments

| Argument | Required | Description | Example |
|-----------|----------|-------------|
| **description** | No | Human-readable description | `"The VPC ID"` |
| **value** | Yes | Computed value | `aws_vpc.main.id` |
| **sensitive** | No | Hide from logs | `true` |
| **precondition** | No | Validation before creation | `precondition { ... }` |
| **depends_on** | No | Output dependencies | `depends_on = [module.vpc]` |

### Output Examples

```hcl
# main.tf
terraform {
  required_version = ">= 1.5.0"
}

variable "region" {
  type    = string
  default = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, 0)

  tags = {
    Name = "public-subnet-1"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "web-server"
  }
}

# Outputs
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "The CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "instance_id" {
  description = "The ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "The public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "instance_private_ip" {
  description = "The private IP address of the EC2 instance"
  value       = instance.web.id
}
```

### Accessing Outputs

```bash
# Display outputs after apply
terraform output

# JSON output (for scripting)
terraform output -json

# Get specific output
terraform output instance_id

# Output example:
# instance_id = "i-1234567890abcdef0"
# instance_public_ip = "54.123.45.67"
```

### Output from Modules

```hcl
# root main.tf calls module
module "vpc" {
  source = "./modules/vpc"
}

# Access module outputs
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "subnet_ids" {
  value = module.vpc.subnet_ids
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}
```

### Conditional Outputs

```hcl
# Conditional output based on count
output "instance_ips" {
  description = "Instance public IPs"
  value       = var.create_instance ? aws_instance.web[*].public_ip : []
}

output "has_instances" {
  description = "Whether instances exist"
  value       = var.create_instance && var.instance_count > 0
}
```

### Sensitive Outputs

```hcl
# Sensitive output for secrets
output "database_password" {
  description = "Database connection password (sensitive)"
  value       = random_password.db_password.result
  sensitive   = true  # Hides from logs
}

# Use sensitive output in resources
resource "aws_db_instance" "main" {
  password = random_password.db_password.result  # Reference sensitive output

  # Don't log password in user_data or tags
  user_data = <<-EOT
    #!/bin/bash
    echo "Database initialized"
  EOT
}
```

### Outputs as Maps

```hcl
# Map of instance IPs by availability zone
output "instance_ips_by_az" {
  description = "Instance IPs mapped by availability zone"
  value = {
    us-east-1a = aws_instance.web_az1.public_ip
    us-east-1b = aws_instance.web_az2.public_ip
    us-east-1c = aws_instance.web_az3.public_ip
  }
}
```

### Output Expressions

```hcl
# Output with concatenation
output "instance_dns_names" {
  description = "Instance DNS names"
  value = "${aws_instance.web_az1.private_dns}.${aws_instance.web_az2.private_dns}"
}

# Output with function call
output "vpc_size" {
  description = "Total VPC IP space (CIDR block range)"
  value       = cidrhost(var.vpc_cidr, 0, 0) - cidrhost(var.vpc_cidr, 0, 0, 0)
}
```

## Best Practices

```hcl
# ✅ DO: Provide descriptions
output "instance_id" {
  description = "EC2 instance identifier"
  value       = aws_instance.web.id
}

# ✅ DO: Use meaningful names
output "api_endpoint" {
  description = "API endpoint for accessing the service"
}

# ❌ DON'T: Use cryptic names
output "id" {  # Bad - not descriptive
}

# ✅ DO: Output only necessary values
output "instance_id" {
  value = aws_instance.web.id  # Required for other tools
}

# ❌ DON'T: Output everything
output "full_instance" {  # Too much, resource exists
  value = aws_instance.web
}

# ✅ DO: Use sensitive for secrets
output "api_key" {
  sensitive = true
  value     = var.api_key
}

# ❌ DON'T: Output sensitive data unmarked
output "password" {
  value = var.password  # Visible in logs!
}
```

### Complete Example

```hcl
# variables.tf
terraform {
  required_version = ">= 1.5.0"
}

variable "project_name" {
  type        = string
  default     = "my-app"
}

variable "environment" {
  type        = string
  default     = "production"
}

variable "create_database" {
  type        = bool
  default     = true
}

# main.tf
module "vpc" {
  source = "./modules/vpc"

  vpc_cidr = "10.0.0.0/16"
}

module "database" {
  source = "./modules/database"
  enabled  = var.create_database
}

module "application" {
  source = "./modules/app"
  vpc_id     = module.vpc.vpc_id
  db_endpoint = module.database.endpoint
}

# outputs.tf
output "vpc_id" {
  description = "VPC ID created by VPC module"
  value       = module.vpc.vpc_id
}

output "db_endpoint" {
  description = "Database endpoint (created by database module)"
  value       = var.create_database ? module.database.endpoint : null
}

output "app_url" {
  description = "Application URL (depends on database)"
  value       = var.create_database ? "https://app.example.com" : null
}
```

## Interview Questions

**Q: What are Terraform outputs and why use them?**
**A:** Outputs return values from configuration for use in other Terraform configs or external tools. Use cases: 1) Module outputs (expose VPC ID to application module), 2) Resource attributes (expose instance IDs to other resources), 3) Communication (provide connection strings), 4) Automation (pass to scripts/CI/CD). Enable code reusability and communication.

**Q: What's the difference between output `description` and `sensitive` flags?**
**A:** `description` is for human-readable documentation about what output represents. `sensitive = true` hides the output value from CLI logs (`terraform output`) and Terraform Cloud UI. State file still contains encrypted value. Use `sensitive` for passwords, API keys, connection strings, or any confidential data.

**Q: How do you access outputs from another Terraform module?**
**A:** Use module output syntax: `output "name" { value = module.module_name.output_name }`. Module must expose the output in its `outputs.tf` file. Root configuration calls module, then references output like: `module.module_name.output_name`.

**Q: Can you reference outputs within the same Terraform configuration?**
**A:** Yes, outputs can be referenced by any resource or data source in same configuration using `output.name.value` syntax. Use for dependent resources or to concatenate values. Example: `subnet_id = module.vpc.public_subnet_ids[0]`.

**Q: What happens to outputs when you run `terraform destroy`?**
**sensitive = false**: Outputs still accessible after destroy (value references fail). **sensitive = true**: Outputs still accessible but hidden. State file marks resources as destroyed, but outputs can still be queried. Outputs are removed from state only when resource definitions are removed.

**Q: How do you output values for use in shell scripts or other tools?**
**A:** Use `terraform output -json` to get machine-readable JSON. Use `terraform output name` to get specific output. Parse with `jq`: `terraform output -json | jq -r '.instance_id.value'`. Or use `terraform output -json` > outputs.json` for file-based workflows.

**Q: What are best practices for Terraform outputs?**
**A:** Best practices: 1) Provide clear descriptions (what the output is, how it's used), 2) Use meaningful names (resource_name_attribute vs generic "id"), 3) Output only necessary values (don't output entire resources), 4) Mark sensitive outputs (`sensitive = true`) for secrets, 5) Group related outputs logically, 6) Keep outputs stable (avoid frequent changes that break consumers), 7) Document required inputs for outputs.

**Q: Can you have multiple outputs with the same name?**
**A:** No, Terraform requires output names to be unique within a configuration. Having duplicate names causes errors. Different modules can have outputs with same name since they're isolated. Use module namespacing if needed: `module.vpc.vpc_id`.

**Q: How do conditional outputs work in Terraform?**
**A:** Conditional outputs use ternary operators or expressions to determine value at apply time. Examples: `value = var.create_resource ? resource.id : null` or `value = length(resources) > 0 ? resources[0].id : ""`. Outputs must be computed, not referenced from resources directly (would cause dependency cycle).

**Q: What happens to outputs when using `terraform plan -refresh-only`?**
**A:** `terraform plan -refresh-only` checks for infrastructure drift. Outputs are still computed from state (not by querying provider). Outputs show current state values, not refreshed values. For refresh, use `terraform refresh` then check outputs separately.

**Q: How do you pass outputs between Terraform and other tools (e.g., Ansible, Kubernetes)?**
**A:** Methods: 1) Write outputs to file (`terraform output -json > outputs.json`), 2) Use `terraform output` in scripts and parse with jq, 3) Terraform Cloud exposes outputs via API, 4) Ansible's Terraform module reads outputs from state, 5) Store in parameter stores (AWS SSM, Vault). Choose method based on integration requirements.

**Q: What's the `precondition` block in Terraform outputs?**
**A:** `precondition` validates that a condition is true BEFORE resource creation or update. If false, plan fails with error message. Use cases: 1) Require specific variables are set (`var.vpc_id != ""`), 2) Verify external dependencies exist (`data.external_resource.id != ""`), 3) Validate configuration is ready. Prevents bad plans from executing.

**Q: Can outputs depend on resources or modules?**
**A:** Yes, outputs can have `depends_on` meta-argument. This ensures outputs are only created after dependencies are ready. Example: `output "app_url" { depends_on = [module.database] }` waits for database to be created before outputting application URL. Useful for cross-module dependencies.

**Q: What's the difference between output expressions and computed locals?**
**A:** Outputs: defined in `output` blocks, persist to state, accessible to other configs. Locals: defined in `locals` blocks, computed at plan time, not persisted to state, only accessible within same configuration. Use outputs for values needed elsewhere, locals for temporary values.

**Q: How do you use outputs in CI/CD pipelines?**
**A:** In CI/CD: 1) `terraform output -json > outputs.json`, 2) Parse specific values with `jq -r '.instance_id.value'`, 3) Store in pipeline artifacts or step outputs for other jobs, 4) Use outputs to trigger next steps in pipeline (e.g., kubectl apply with outputted values). Terraform Cloud also provides outputs via API for automation.

**Q: What happens when you rename or remove an output?**
**A:** Renaming or removing outputs breaks consumers that reference them. Terraform shows warning but doesn't stop you. Plan will show resources will be recreated or destroyed to remove output references. Carefully update all consumers when changing outputs to avoid cascading failures.

**Q: Can outputs reference each other (circular dependencies)?**
**A:** Terraform allows circular output references as long as there's a way to compute all values. However, this can be fragile. Example: Output A references B, Output B references A. Plan succeeds but changes to one affects the other. Avoid circular dependencies when possible.

**Q: How does the `sensitive` flag affect Terraform CLI and state storage?**
**A:** `sensitive = true` hides the value from: 1) CLI output (`terraform output` shows `(sensitive value)`), 2) Terraform Cloud UI (shows `<sensitive>`), 3) Logs and state file. State file stores encrypted value but doesn't decrypt it in logs. CLI and API never expose raw value to unauthenticated callers.

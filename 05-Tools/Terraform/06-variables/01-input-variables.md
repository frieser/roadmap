---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Input Variables

## Summary

**Input variables** in Terraform allow you to parameterize configurations, making them reusable and dynamic. Defined with `variable` blocks, variables accept values from CLI flags, `.tfvars` files, environment variables, or default values. Variables support type constraints, descriptions, and validation rules, ensuring configuration integrity across environments (dev, staging, prod).

## Detailed Explanation

### Defining Variables

```hcl
# Basic variable definition
variable "region" {
  type    = string
  default = "us-east-1"
}

# Variable with description
variable "project_name" {
  description = "Project name used for resource naming"
  type        = string
  default     = "my-app"
}

# Variable with no default (required)
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}
```

### Variable Sources (Priority Order)

```mermaid
graph TB
    A[Variable Value] --> B{Source Priority}

    B --> C1[Environment Variables]
    B --> C2[CLI -auto-approve -var]
    B --> C3[.tfvars Files]
    B --> C4[*.auto.tfvars Files]
    B --> C5[Default Value]

    style C1 fill:#e1ffe1
    style C2 fill:#e1f5ff
    style C3 fill:#ffe1e1
    style C4 fill:#fff4e1
    style C5 fill:#e1f5ff
```

| Priority | Source | Example |
|----------|---------|---------|
| **1** | CLI `-var` flag | `terraform apply -var='region=us-east-1'` |
| **2** | `*.auto.tfvars` files | Loaded automatically (alphabetical order) |
| **3** | `terraform.tfvars` file | Loaded automatically if present |
| **4** | Environment variables | `TF_VAR_region=us-east-1` |
| **5** | Variable default value | `default = "value"` |

### CLI Variables

```bash
# Single variable
terraform apply -var="region=us-east-1"

# Multiple variables
terraform apply \
  -var="region=us-east-1" \
  -var="environment=production" \
  -var="instance_count=3"

# Variable with map
terraform apply -var='instance_sizes={"dev":"t3.micro","prod":"t3.large"}'

# Variable from file
terraform apply -var-file=prod.tfvars

# Multiple variable files
terraform apply \
  -var-file=common.tfvars \
  -var-file=prod.tfvars
```

### Variable Definition Files

```bash
# terraform.tfvars
region        = "us-east-1"
environment   = "production"
instance_count = 3
instance_type = "t3.large"

# prod.tfvars
environment   = "production"
instance_count = 5
instance_type = "t3.large"

# dev.tfvars
environment   = "development"
instance_count = 1
instance_type = "t3.micro"
```

```hcl
# variables.tf
variable "environment" {
  type    = string
  default = "dev"
}

variable "instance_count" {
  type    = number
  default = 1
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# main.tf
resource "aws_instance" "web" {
  count         = var.instance_count
  instance_type = var.instance_type

  tags = {
    Environment = var.environment
  }
}
```

### Environment Variables

```bash
# Export Terraform variables
export TF_VAR_region="us-east-1"
export TF_VAR_environment="production"
export TF_VAR_instance_count="3"

# Terraform will automatically recognize these
terraform apply

# Works with all commands
terraform plan
terraform apply
```

### Auto-loaded Variables

```bash
# Files with .auto.tfvars suffix are auto-loaded
terraform.tfvars        # Auto-loaded
terraform.auto.tfvars   # Auto-loaded (lower priority)
prod.auto.tfvars       # Auto-loaded
override.auto.tfvars    # Auto-loaded (higher priority)

# Priority (highest to lowest):
# 1. CLI -var flags
# 2. *.auto.tfvars files (alphabetical)
# 3. terraform.tfvars (if present)
# 4. Environment variables
# 5. Variable defaults

# Example file structure:
#/
#  main.tf
#  variables.tf
#  dev.auto.tfvars    # Auto-loaded
#  prod.auto.tfvars   # Auto-loaded
#  terraform.tfvars    # Manual load
```

### Complex Variable Types

```hcl
# String variable
variable "name" {
  type    = string
  default = "my-app"
}

# Number variable
variable "port" {
  type    = number
  default = 8080
}

# Boolean variable
variable "enabled" {
  type    = bool
  default = true
}

# List variable
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Map variable
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Object variable
variable "instance_config" {
  type = object({
    ami           = string
    instance_type = string
    volume_size   = number
  })
  default = {
    ami           = "ami-0c55b159cbfafe1f0"
    instance_type = "t3.micro"
    volume_size   = 20
  }
}

# Tuple variable
variable "ports" {
  type = tuple([number, number, number])
  default = [80, 443, 8080]
}

# Set variable
variable "security_groups" {
  type    = set(string)
  default = ["sg-001", "sg-002"]
}
```

### Using Variables in Configuration

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Simple reference
resource "aws_instance" "web" {
  ami = "ami-123"
  tags = {
    Environment = var.environment
  }
}

# Variable in expression
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = var.instance_types[var.environment]
}

# Variable in conditional
resource "aws_instance" "web" {
  monitoring = var.environment == "prod" ? true : false
}

# Variable in string interpolation
resource "aws_s3_bucket" "logs" {
  bucket = "${var.project_name}-${var.environment}-logs"
}
```

### Sensitive Variables

```hcl
variable "database_password" {
  type      = string
  sensitive = true  # Hides from output
}

# Usage
resource "aws_db_instance" "main" {
  password = var.database_password
}

# Outputting sensitive variable
output "db_connection_string" {
  sensitive = true
  value     = "postgres://user:${var.database_password}@host:5432/db"
}
```

### Variable Validation

```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"

  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must start with 't3.'"
  }
}

variable "port_number" {
  type        = number
  description = "Port number for application"

  validation {
    condition     = var.port_number >= 1 && var.port_number <= 65535
    error_message = "Port number must be between 1 and 65535"
  }
}

variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}
```

### Environment-Specific Configuration

```hcl
# variables.tf
variable "environment" {
  type    = string
  default = "dev"
}

variable "config" {
  type = map(object({
    instance_count = number
    instance_type = string
    monitoring    = bool
  }))
  default = {
    dev = {
      instance_count = 1
      instance_type = "t3.micro"
      monitoring    = false
    }
    staging = {
      instance_count = 2
      instance_type = "t3.small"
      monitoring    = true
    }
    prod = {
      instance_count = 5
      instance_type = "t3.large"
      monitoring    = true
    }
  }
}

# main.tf
locals {
  env_config = var.config[var.environment]
}

resource "aws_instance" "web" {
  count         = local.env_config.instance_count
  instance_type = local.env_config.instance_type
  monitoring    = local.env_config.monitoring
}
```

```bash
# dev.auto.tfvars
environment = "dev"

# staging.auto.tfvars
environment = "staging"

# prod.auto.tfvars
environment = "prod"

# Apply specific environment
terraform apply -var-file=prod.auto.tfvars
```

### Best Practices

```hcl
# ✅ Do: Provide descriptions
variable "region" {
  description = "AWS region for resources"
  type        = string
}

# ❌ Don't: Omit descriptions
variable "region" {
  type = string
}

# ✅ Do: Use type constraints
variable "subnets" {
  type = list(string)
}

# ❌ Don't: Use untyped variables
variable "subnets" {
  # Any type accepted - bad practice
}

# ✅ Do: Set sensible defaults
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# ✅ Do: Validate inputs
variable "cidr_block" {
  type = string

  validation {
    condition     = can(cidrhost(var.cidr_block))
    error_message = "Must be a valid CIDR block"
  }
}

# ✅ Do: Organize variable files
# terraform.tfvars   - Common defaults
# prod.tfvars       - Production overrides
# dev.auto.tfvars   - Dev environment
```

### Complete Example

```hcl
# variables.tf
variable "project_name" {
  description = "Project name used for resource naming"
  type        = string
  default     = "my-application"
}

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_count" {
  description = "Number of EC2 instances to create"
  type        = number
  default     = 1

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 100
    error_message = "Instance count must be between 1 and 100"
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must be t3.* family"
  }
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Additional tags for all resources"
  type        = map(string)
  default     = {}
}

# main.tf
locals {
  common_tags = merge(var.tags, {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  })
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags       = local.common_tags
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  monitoring    = var.enable_monitoring

  tags = merge(local.common_tags, {
    Name = "${var.project_name}-web-${count.index}"
  })
}
```

```bash
# prod.tfvars
environment       = "production"
instance_count    = 5
instance_type     = "t3.large"
enable_monitoring = true

tags = {
  Owner   = "team-platform"
  CostCode = "EC-001"
}

# Apply with overrides
terraform apply -var-file=prod.tfvars
```

## Interview Questions

**Q: What are input variables in Terraform and why use them?**
**A:** Input variables allow you to parameterize Terraform configurations, making them reusable and dynamic. Instead of hard-coding values like `instance_type = "t3.micro"`, you use `instance_type = var.instance_type`. This lets you use same configuration for dev, staging, prod by just changing variable values.

**Q: What are the different ways to provide variable values to Terraform?**
**A:** Variables can be provided: 1) CLI `-var` flag: `terraform apply -var="region=us-east-1"`, 2) Variable definition files (`*.tfvars`): `terraform apply -var-file=prod.tfvars`, 3) Environment variables: `export TF_VAR_region=us-east-1`, 4) Default values in variable block: `default = "value"`. Priority: CLI > environment > .tfvars > default.

**Q: What's the difference between `terraform.tfvars` and `terraform.auto.tfvars`?**
**A:** `terraform.tfvars` IS auto-loaded by Terraform if present in the current directory. `terraform.auto.tfvars` (any file ending in `.auto.tfvars`) is also auto-loaded but has a higher precedence. Use `.auto.tfvars` for environment-specific configs you want loaded automatically, and `-var-file` if you want to explicitly pass a specific file.

**Q: How do you define and use complex variable types in Terraform?**
**A:** Complex types include: `list(string)` for ordered collections, `map(string)` for key-value pairs, `set(string)` for unique values, `object({})` for structured data, `tuple([...])` for fixed-length typed lists. Example: `variable "config" { type = object({name = string, count = number}) }`.

**Q: What are variable validations in Terraform and when would you use them?**
**A:** Variable validations enforce constraints on input values using `validation` blocks. Use them to catch errors early: check ranges (`count >= 1`), validate formats (`regex("^t3\\.", type)`), or restrict to allowed values (`contains(["dev", "prod"], env)`). Validation runs at plan time, before any resources are created.

**Q: How do you handle sensitive variables in Terraform?**
**A:** Mark variables as `sensitive = true` in the variable definition. This hides the value from logs, UI output, and state files (stored encrypted). When outputting sensitive values, also mark output as `sensitive = true`. Use environment variables or secret management for actual secrets (don't commit to git).

**Q: Can you use environment variables to set Terraform variables?**
**A:** Yes, Terraform automatically recognizes environment variables prefixed with `TF_VAR_`. For example, `export TF_VAR_region=us-east-1` sets the `region` variable. Works for all Terraform commands (plan, apply, etc.). Useful for secrets or CI/CD environments.

**Q: How do you create environment-specific configurations using variables?**
**A:** Create variable files for each environment: `dev.auto.tfvars`, `staging.auto.tfvars`, `prod.auto.tfvars`. Each file sets environment-specific values. Run `terraform apply` in each environment's directory, or use workspace/variable pattern: `terraform workspace select prod` then `terraform apply`.

**Q: What's the purpose of variable descriptions?**
**A:** Variable descriptions document what the variable does and expected format. They appear in `terraform -help` output, Terraform Cloud UI, and help teammates understand required inputs. Example: `description = "AWS region for resources"`. Good practice: always provide descriptions for non-obvious variables.

**Q: How does Terraform resolve variable values when multiple sources are available?**
**A:** Priority order (highest to lowest): 1) CLI `-var` flags, 2) `.auto.tfvars` files, 3) `terraform.tfvars`, 4) Environment variables (`TF_VAR_*`), 5) Variable default values.

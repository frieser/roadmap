---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Creating Local Modules

## Summary

**Local Terraform modules** are reusable configurations stored in the same repository as your root module. They use `source = "./modules/name"` syntax, enabling code reuse, version control via Git, and easy sharing within team. Local modules should follow structure with `main.tf`, `variables.tf`, and `outputs.tf` for clarity and maintainability.

## Detailed Explanation

### Creating a Module Structure

```bash
# Standard module directory structure
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── ec2/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
```

### Module Files

```hcl
# modules/vpc/main.tf - Define resources
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags       = var.tags
}

resource "aws_subnet" "public" {
  count      = length(var.availability_zones)
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, count.index)
}

# modules/vpc/variables.tf - Define inputs
variable "vpc_cidr" {
  type        = string
  description = "CIDR block for VPC"
}

variable "availability_zones" {
  type        = list(string)
  description = "List of availability zones"
}

variable "tags" {
  type        = map(string)
  description = "Tags to apply to all resources"
  default     = {}
}

# modules/vpc/outputs.tf - Return values
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}
```

### Calling Local Module

```hcl
# main.tf (root module)
module "vpc" {
  source = "./modules/vpc"

  # Pass inputs
  vpc_cidr        = var.vpc_cidr
  availability_zones = var.availability_zones
  tags             = merge(var.default_tags, {
    Environment = var.environment
  })
}

# Access module outputs
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

### Module Documentation

```markdown
# modules/vpc/README.md
# VPC Module

This module creates a VPC with public subnets in AWS.

## Usage

```hcl
module "vpc" {
  source = "./modules/vpc"
  vpc_cidr = "10.0.0.0/16"
}
```

## Inputs

| Name | Type | Default | Description |
|-------|-------|----------|-------------|
| vpc_cidr | string | - | CIDR block for VPC |
| availability_zones | list(string) | - | List of availability zones |
| tags | map(string) | {} | Tags to apply |

## Outputs

| Name | Description |
|-------|-------------|
| vpc_id | VPC ID |
| public_subnet_ids | List of public subnet IDs |
```

### Module Best Practices

```hcl
# ✅ DO: Separate concerns
# Each module should do one thing well
# module/vpc - networking only
# module/ec2 - compute only

# ✅ DO: Use descriptive names
# Good: modules/web-server, modules/database
# Bad: modules/stuff, modules/utils

# ✅ DO: Document all inputs/outputs
variable "instance_type" {
  type        = string
  description = "EC2 instance type (t3.micro, t3.small, t3.medium)"
  default     = "t3.micro"
}

output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

# ❌ DON'T: Hard-code values
# Bad:
# ami = "ami-123456"

# ✅ DO: Pass as variables
# Good:
# ami = var.ami

# ❌ DON'T: Assume defaults
# Document required vs optional
variable "required" {
  type = string  # No default = required
}

variable "optional" {
  type    = string
  default = "value"  # Has default = optional
}
```

### Complete Example

```hcl
# modules/ec2/main.tf
variable "instance_count" {
  type    = number
  default = 1
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "ami" {
  type    = string
}

variable "subnet_id" {
  type    = string
}

variable "tags" {
  type    = map(string)
  default = {}
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = var.ami
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  tags          = merge(var.tags, {
    Name = "web-${count.index}"
  })
}

output "instance_ids" {
  value = aws_instance.web[*].id
}
```

```hcl
# modules/ec2/variables.tf
variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 1
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = can(regex("^t[23]\\.(nano|micro|small|medium|large|xlarge|2xlarge)", var.instance_type))
    error_message = "Must be a valid T2/T3 instance type."
  }
}

variable "ami" {
  description = "AMI ID for instances"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID to launch instances in"
  type        = string
}

variable "tags" {
  description = "Additional tags for instances"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/ec2/outputs.tf
output "instance_ids" {
  description = "IDs of created instances"
  value       = aws_instance.web[*].id
}

output "instance_public_ips" {
  description = "Public IP addresses of instances"
  value       = aws_instance.web[*].public_ip
}

output "instance_private_ips" {
  description = "Private IP addresses of instances"
  value       = aws_instance.web[*].private_ip
}
```

```hcl
# main.tf (root)
module "vpc" {
  source = "./modules/vpc"

  vpc_cidr        = var.vpc_cidr
  availability_zones = var.availability_zones
}

module "ec2" {
  source     = "./modules/ec2"
  depends_on = [module.vpc]

  ami           = var.ami
  instance_type = var.instance_type
  instance_count = var.instance_count
  subnet_id     = module.vpc.public_subnet_ids[0]
  tags          = var.tags
}

output "instance_ips" {
  value = module.ec2.instance_public_ips
}
```

### Module Organization Patterns

```bash
# Pattern 1: By function
modules/
├── networking/
│   ├── vpc/
│   ├── subnet/
│   └── security-groups/
├── compute/
│   ├── ec2/
│   └── lambda/
└── storage/
    ├── s3/
    └── rds/

# Pattern 2: By component
modules/
├── vpc/
├── load-balancer/
├── auto-scaling/
└── monitoring/

# Pattern 3: By layer
modules/
├── base/
├── services/
└── applications/
```

### Testing Local Modules

```bash
# Test module in isolation
cd modules/vpc

# Create test variables
cat > test.tfvars << EOF
vpc_cidr = "10.99.0.0/16"
availability_zones = ["us-east-1a"]
EOF

# Plan to verify
terraform plan -var-file=test.tfvars

# Clean up test state
rm -rf .terraform terraform.tfstate terraform.tfstate.backup
```

### Sharing Local Modules

```bash
# Option 1: Git repository
# Commit and push module changes
git add modules/
git commit -m "Update VPC module"
git push origin main

# Option 2: Symlink to shared location
ln -s ../shared-modules/vpc ./modules/vpc

# Option 3: Git submodule
git submodule add https://github.com/org/terraform-modules.git modules/shared
```

## Interview Questions

**Q: How do you create a local Terraform module?**
**A:** Create a directory with `.tf` files (`main.tf`, `variables.tf`, `outputs.tf`). Define resources in `main.tf`, inputs in `variables.tf`, outputs in `outputs.tf`. Call from root using `module "name" { source = "./modules/name" }` syntax. All `.tf` files in module directory are automatically loaded.

**Q: What files should a Terraform module contain?**
**A:** Standard module structure: `main.tf` (resources, data sources, locals), `variables.tf` (input parameters with type, description, default), `outputs.tf` (return values). Optional: `README.md` (documentation), `versions.tf` (version constraints).

**Q: How do you pass inputs to a local module?**
**A:** Pass inputs as block arguments when calling module: `module "vpc" { vpc_cidr = "10.0.0.0/16" }`. Arguments must match input variables defined in child module's `variables.tf`. Can use expressions, variables, or values.

**Q: How do you access outputs from a local module?**
**A:** Access outputs using `<module_name>.<output_name>` syntax: `output "vpc_id" { value = module.vpc.vpc_id }`. Root can use module outputs in expressions or pass them to other modules. Outputs enable composition of modules.

**Q: What's the difference between local and remote modules?**
**A:** Local modules: `source = "./modules/vpc"` - stored in same repository, managed by Git, updated with pull. Remote modules: `source = "terraform-aws-modules/vpc/aws"` - stored externally (Terraform Registry, GitHub), downloaded by Terraform, version-controlled with tags/branches.

**Q: How do you document Terraform modules?**
**A:** Use `README.md` in module directory with sections: Description, Usage, Inputs (table with type/description/default), Outputs. Document all variables with descriptions. Provide example usage showing how to call module with inputs.

**Q: What are best practices for organizing local Terraform modules?**
**A:** Best practices: 1) Single responsibility (one function per module), 2) Descriptive naming (`modules/vpc`, not `modules/stuff`), 3) Separate files (`main.tf`, `variables.tf`, `outputs.tf`), 4) Document inputs/outputs, 5) Validate inputs, 6) Test modules in isolation.

**Q: Can Terraform modules have child modules (nested)?**
**A:** Yes, modules can nest indefinitely. Child module can call other child modules (grandchildren). Useful for layered architecture: root calls `infrastructure` module, which calls `vpc`, `ec2`, `security` modules. Outputs bubble up through module chain.

**Q: How do you version control local Terraform modules?**
**A:** Store in Git repository. Each commit is a version. Use semantic versioning in tags (v1.0.0, v1.0.1). Root module pins module version or uses git ref. Updates require git pull. Module source: `source = "./modules/vpc"`.

**Q: What's the benefit of separating module files (main.tf, variables.tf, outputs.tf)?**
**A:** Separation improves organization: `variables.tf` defines interface, `main.tf` implements logic, `outputs.tf` defines what's returned. Easier navigation, understanding what module does, and maintenance. All `.tf` files are still loaded together.

**Q: How do you test local Terraform modules before using them?**
**A:** Create test configuration in module directory with test variables. Run `terraform init`, `terraform plan -var-file=test.tfvars` to verify module works. Check for validation errors, plan correctness. Clean up test state (`rm .terraform/`) after testing.

**Q: Can local modules define their own providers?**
**A:** Typically no - providers inherited from root module. Best practice: configure providers once in root. Exception: explicit provider passing for multi-region/multi-account: `module "vpc" { providers = { aws = aws.west } }`. Most modules rely on root provider configuration.

---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Variable Type Constraints

## Summary

**Type constraints** in Terraform variables enforce data types and values, catching errors before infrastructure changes. Terraform supports primitive types (string, number, bool), complex types (list, map, set, object, tuple), and validation rules for each. Type safety prevents misconfiguration and enables autocomplete in IDEs.

## Detailed Explanation

### Primitive Types

```hcl
# String type
variable "region" {
  type        = string
  description = "AWS region"
  default     = "us-east-1"
}

# Number type
variable "port" {
  type        = number
  description = "Application port"
  default     = 8080
}

# Boolean type
variable "enabled" {
  type        = bool
  description = "Enable feature flag"
  default     = false
}
```

### Complex Types

```hcl
# List type
variable "availability_zones" {
  type        = list(string)
  description = "List of availability zones"
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Map type
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Set type
variable "security_groups" {
  type    = set(string)
  default = ["sg-001", "sg-002", "sg-003"]
}

# Object type
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

# Tuple type
variable "ports" {
  type = tuple([number, number, number])
  default = [80, 443, 8080]
}
```

### Optional vs Required

```hcl
# Required variable (no default)
variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR block (required)"
}

# Optional variable (with default)
variable "enable_dns" {
  type        = bool
  default     = false
  description = "Enable DNS (optional)"
}

# Using optional variable in resource
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr  # Required - must be provided
}
```

### Type Validation

```hcl
# String validation with regex
variable "instance_type" {
  type        = string
  description = "EC2 instance type"

  validation {
    condition     = can(regex("^t[23]\\.(nano|micro|small|medium|large|xlarge|2xlarge)", var.instance_type))
    error_message = "Instance type must be t2 or t3 family."
  }
}

# Number validation with range
variable "instance_count" {
  type        = number
  description = "Number of instances"

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 100
    error_message = "Instance count must be between 1 and 100."
  }
}

# List validation with length
variable "subnets" {
  type        = list(string)
  description = "List of subnet CIDRs"

  validation {
    condition     = length(var.subnets) >= 1 && length(var.subnets) <= 10
    error_message = "Subnet list must have 1-10 entries."
  }
}

# CIDR validation
variable "cidr_block" {
  type        = string

  validation {
    condition     = can(cidrhost(var.cidr_block))
    error_message = "Must be a valid CIDR block."
  }
}

# Allowed values validation
variable "environment" {
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Complex Type Examples

```hcl
# Map of objects
variable "servers" {
  type = map(object({
    ami           = string
    instance_type = string
    count         = number
  }))
  default = {
    web = {
      ami           = "ami-001"
      instance_type = "t3.micro"
      count         = 1
    }
    db = {
      ami           = "ami-002"
      instance_type = "t3.medium"
      count         = 1
    }
  }
}

# Nested objects
variable "advanced_config" {
  type = object({
    base = object({
      ami           = string
      instance_type = string
    })
    overrides = map(object({
      instance_type = string
    }))
  })
  default = {
    base = {
      ami           = "ami-123"
      instance_type = "t3.micro"
    }
    overrides = {}
  }
}
```

### Type Coercion

```hcl
# String to list
variable "zones" {
  type    = string
  default = "us-east-1a,us-east-1b,us-east-1c"
}

# Convert string to list
locals {
  availability_zones = split(",", var.zones)
}

# Number to string
variable "port" {
  type    = number
  default = 8080
}

output "port_string" {
  value = tostring(var.port)
}
```

### Default Values

```hcl
# Default values should be sensible
variable "instance_type" {
  type        = string
  default     = "t3.micro"  # Reasonable default
  description = "EC2 instance type"
}

variable "monitoring" {
  type        = bool
  default     = false  # Security default (disable by default)
  description = "Enable detailed monitoring (additional cost)"
}

# Default empty for collections
variable "tags" {
  type    = map(string)
  default = {}  # Empty map allows optional tagging
}
```

### Type Constraints Summary

| Type | Syntax | Example | Use Case |
|-------|---------|---------|-----------|
| **string** | `type = string` | `"hello"` | Text values |
| **number** | `type = number` | `42`, `3.14` | Counts, sizes |
| **bool** | `type = bool` | `true`, `false` | Flags, toggles |
| **list** | `type = list(type)` | `["a", "b"]` | Ordered collections |
| **map** | `type = map(type)` | `{key = value}` | Key-value pairs |
| **set** | `type = set(type)` | `["a", "b"]` | Unique values |
| **object** | `type = object({...})` | `{name = val}` | Structured data |
| **tuple** | `type = tuple([...])` | `[string, number]` | Fixed-type list |

### Best Practices

```hcl
# ✅ DO: Always specify type constraints
variable "name" {
  type = string  # Explicit type
}

# ❌ DON'T: Leave untyped
variable "name" {
  # Any type accepted - bad practice
}

# ✅ DO: Provide sensible defaults
variable "instance_type" {
  type    = string
  default = "t3.micro"  # Safe default
}

# ✅ DO: Validate inputs
variable "cidr_block" {
  type = string

  validation {
    condition     = can(cidrhost(var.cidr_block))
    error_message = "Invalid CIDR block."
  }
}

# ❌ DON'T: Use defaults for sensitive data
variable "password" {
  type = string
  default = "secret123"  # Never default secrets!
}

# ✅ DO: Use descriptions
variable "instance_count" {
  type        = number
  default     = 1
  description = "Number of EC2 instances to deploy (1-100)"
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
  description = "Project name used for resource naming"
}

variable "environment" {
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vpc_cidr" {
  type        = string
  default     = "10.0.0.0/16"

  validation {
    condition     = can(cidrhost(var.vpc_cidr))
    error_message = "Must be a valid CIDR block."
  }
}

variable "instance_count" {
  type        = number
  default     = 1

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

variable "tags" {
  type    = map(string)
  default = {}
}

locals {
  common_tags = merge(var.tags, {
    Project     = var.project_name
    Environment = var.environment
  })
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags       = local.common_tags
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_types[var.environment]
  subnet_id     = aws_subnet.public.id

  tags = merge(local.common_tags, {
    Name = "${var.project_name}-${var.environment}-web-${count.index}"
  })
}
```

## Interview Questions

**Q: What are type constraints in Terraform and why use them?**
**A:** Type constraints enforce specific data types (string, number, bool, list, map, etc.) on variables. They catch configuration errors early (at plan time), enable IDE autocomplete, and make code more maintainable. Without type constraints, any value is accepted.

**Q: What are the main variable types supported by Terraform?**
**A:** Primitive types: string, number, bool. Complex types: list (ordered collection), map (key-value pairs), set (unique unordered collection), object (structured data with named attributes), tuple (fixed-length typed list). Each can be parameterized: `list(string)`, `map(number)`.

**Q: How do you create a required variable vs an optional variable?**
**A:** Required variable has no `default` value - user must provide it: `variable "name" { type = string }`. Optional variable has `default` value - module works without input: `variable "name" { type = string, default = "default-value" }`. Use required for essential parameters, optional for enhancements.

**Q: What is the difference between list and set in Terraform?**
**A:** List is ordered collection that allows duplicate values. Set is unordered collection that automatically removes duplicates. Use list when order matters or duplicates acceptable. Use set when order doesn't matter and uniqueness is important (e.g., security groups).

**Q: How do you validate variable values in Terraform?**
**A:** Use `validation` block within variable definition. Supported checks: `condition` with expressions like `length(var.list) >= 1`, `can(regex("pattern", var.value))`, `contains([...], var.value)`, `can(cidrhost(var.cidr))`. Validation runs at plan time and shows error message if condition fails.

**Q: What is an object type in Terraform variables?**
**A:** Object is structured data type with named attributes: `object({ name = string, count = number })`. Useful for grouping related parameters. Access attributes with dot notation: `var.config.name`, `var.config.count`. Groups related values into single variable.

**Q: How do you create a map of objects in Terraform?**
**A:** Use `map(object({...}))` type: `variable "servers" { type = map(object({ami = string, type = string})) }`. Each key maps to an object with those attributes. Access with `var.servers.web.ami`, `var.servers.db.type`. Useful for configuration dictionaries.

**Q: What is the difference between object and map in Terraform?**
**A:** Object is a fixed structure with specific attributes: `object({name = string, count = number})`. Map is a dynamic key-value collection: `map(string)`. Object values accessed with dot notation (`obj.attr`), map values accessed with keys (`map["key"]`). Use object for structures, map for dictionaries.

**Q: How does Terraform handle type coercion between types?**
**A:** Terraform automatically converts between compatible types: string ↔ number, bool ↔ string, etc. Manual coercion using functions: `toString()`, `tonumber()`, `tobool()`. Use type constraints to ensure correct type from start, avoid unexpected coercion issues.

**Q: What are best practices for setting variable defaults?**
**A:** Best practices: 1) Provide sensible defaults for non-critical values, 2) No defaults for required inputs (forces user to provide), 3) No defaults for sensitive data (never hard-code secrets), 4) Default to safe/conservative values (e.g., disable expensive features), 5) Document why default exists in description.

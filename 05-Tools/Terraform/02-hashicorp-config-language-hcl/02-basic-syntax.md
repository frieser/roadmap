---
tags: ['terraform', 'hcl', 'tools', 'roadmap']
---

# HCL Basic Syntax

## Summary

HCL syntax is human-readable with blocks, arguments, and expressions. Blocks contain related configuration, arguments are key-value pairs, and expressions use variables, functions, and references. HCL supports comments, strings (quoted), numbers, booleans, lists, maps, and heredocs for multi-line text. File naming uses `.tf` extension, and all `.tf` files in directory are automatically loaded.

## Detailed Explanation

### File Structure

```bash
# .tf files contain HCL configuration
main.tf           # Main resources
variables.tf       # Input variables
outputs.tf        # Output values
provider.tf       # Provider configuration
backend.tf        # State backend config
versions.tf       # Version constraints

# All .tf files are automatically merged
# Multiple files = organization, not required
```

### Comments

```hcl
# Single-line comment

# Comments can start anywhere
resource "aws_instance" "web" {  # After a line

  ami = "ami-123"  # Inline comment
}

/*
  Multi-line comment
  Can span multiple lines
  Useful for documentation
*/
```

### Blocks

```hcl
# Block syntax: block_type "name" { }

# Terraform block - Version constraints
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Provider block - Configure provider
provider "aws" {
  region = "us-east-1"
}

# Resource block - Create infrastructure
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "main-vpc"
  }
}

# Variable block - Input parameters
variable "region" {
  type    = string
  default = "us-east-1"
}

# Output block - Return values
output "vpc_id" {
  value = aws_vpc.main.id
}

# Data block - Query existing resources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Locals block - Local values
locals {
  common_tags = {
    Environment = "production"
  }
}
```

### Arguments

```hcl
# Arguments are key-value pairs within blocks

resource "aws_instance" "web" {
  # Required arguments
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # Optional arguments
  monitoring    = true
  user_data     = <<-EOT
#!/bin/bash
echo "Hello" > /tmp/hello.txt
EOT
}
```

### Strings

```hcl
# Double-quoted strings
name = "web-server"
path = "/var/www/html"

# Template strings (interpolation)
greeting = "Hello, ${var.username}!"
# Or HCL 2: greeting = "Hello, ${var.username}!"

# Escaping special characters
escaped = "This is a \"quote\""

# Multiline strings with heredoc
script = <<-EOT
#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl start nginx
EOT
```

### Numbers

```hcl
# Integers
count = 10
port  = 8080

# Floats
cpu_limit = 0.5
memory    = 2048.5

# Scientific notation
value = 1.5e10
```

### Booleans

```hcl
# Boolean values
enabled = true
disabled = false

# In conditions
monitoring = var.environment == "prod" ? true : false
```

### Lists

```hcl
# List of strings
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

# List of numbers
ports = [80, 443, 8080]

# List of objects
instance_configs = [
  {
    name = "web"
    type = "t3.micro"
  },
  {
    name = "db"
    type = "t3.medium"
  }
]

# List in variable
variable "subnets" {
  type    = list(string)
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}
```

### Maps (Dictionaries)

```hcl
# Map of strings
tags = {
  Name        = "web-server"
  Environment = "production"
  Owner       = "team-platform"
}

# Map of numbers
instance_sizes = {
  small  = 1
  medium = 2
  large  = 4
}

# Map of objects
instance_config = {
  web = {
    ami           = "ami-001"
    instance_type = "t3.micro"
  }
  db = {
    ami           = "ami-002"
    instance_type = "t3.medium"
  }
}

# Map in variable
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Access map values
type = var.instance_types[var.environment]
```

### Sets

```hcl
# Set (unique values, no duplicates)
subnets = [
  "subnet-1",
  "subnet-2",
  "subnet-1"  # Duplicate - will be removed in set
]

# Variable set type
variable "security_groups" {
  type    = set(string)
  default = ["sg-001", "sg-002", "sg-003"]
}
```

### Objects

```hcl
# Object with typed attributes
variable "config" {
  type = object({
    name  = string
    count = number
    tags  = map(string)
  })
  default = {
    name  = "web"
    count = 1
    tags  = {}
  }
}

# Access object attributes
resource "aws_instance" "web" {
  tags = merge(var.config.tags, {
    Name = var.config.name
  })
}
```

### Tuples

```hcl
# Tuple - Fixed-length list with specific types
variable "instance_info" {
  type = tuple([string, number, bool])
  default = ["web", 1, true]
}

# Access tuple elements
name = var.instance_info[0]  # "web"
count = var.instance_info[1]  # 1
```

### Heredocs

```hcl
# Standard heredoc - preserves whitespace
user_data = <<-EOT
#!/bin/bash
echo "Hello World"
apt-get update
EOT

# Indented heredoc - strips leading whitespace
user_data = <<-EOF
#!/bin/bash
echo "Hello World"
apt-get update
EOF

# Heredoc for configuration files
config_data = <<-CONF
[nginx]
user = www-data
port = 80
CONF
```

### Operators

```hcl
# Arithmetic
sum     = 1 + 2
diff    = 10 - 3
product  = 5 * 4
quotient = 20 / 4

# Comparison
equals     = var.a == var.b
not_equal  = var.a != var.b
greater    = var.a > var.b
less       = var.a < var.b
greater_eq = var.a >= var.b
less_eq    = var.a <= var.b

# Logical
and = var.a && var.b
or  = var.a || var.b
not = !var.a

# Ternary (conditional)
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
```

### Variables and References

```hcl
# Define variable
variable "project_name" {
  type    = string
  default = "my-app"
}

# Reference variable
resource "aws_s3_bucket" "data" {
  bucket = "${var.project_name}-data"
}

# Reference resource
resource "aws_instance" "web" {
  ami = "ami-123"
}

resource "aws_eip" "web_eip" {
  instance = aws_instance.web.id  # Reference aws_instance.web
}

# Reference data source
data "aws_vpc" "existing" {
  id = "vpc-123"
}

resource "aws_subnet" "public" {
  vpc_id = data.aws_vpc.existing.id
}

# Reference module outputs
module "vpc" {
  source = "./modules/vpc"
}

resource "aws_subnet" "private" {
  vpc_id = module.vpc.vpc_id
}
```

### Expressions

```hcl
# String interpolation
message = "Server: ${var.server_name} in ${var.region}"

# List indexing
first_subnet = var.subnets[0]
last_subnet  = var.subnets[length(var.subnets) - 1]

# Map lookup
type = var.instance_sizes[var.size]

# Function calls
upper_name   = upper(var.name)
subnet_cidr = cidrsubnet(var.vpc_cidr, 8, count.index)
```

### File Organization Best Practices

```hcl
# main.tf - Primary resources
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags       = local.common_tags
}

# variables.tf - All variables
variable "region" {
  type    = string
  default = "us-east-1"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "common_tags" {
  type    = map(string)
  default = {
    ManagedBy = "Terraform"
  }
}

# outputs.tf - All outputs
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}

# locals.tf - Local values
locals {
  common_tags = merge(var.common_tags, {
    Environment = var.environment
  })
}
```

### Common Syntax Patterns

```hcl
# Pattern 1: Conditional values
resource "aws_instance" "web" {
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
  monitoring  = var.environment == "prod" ? true : false
}

# Pattern 2: Dynamic naming
resource "aws_instance" "web" {
  tags = {
    Name = "${var.project}-${var.environment}-web-${count.index}"
  }
}

# Pattern 3: Tag merging
resource "aws_instance" "web" {
  tags = merge(var.default_tags, {
    Name = "web-server"
    Role = var.role
  })
}

# Pattern 4: List comprehension
resource "aws_subnet" "public" {
  count          = length(var.availability_zones)
  vpc_id         = aws_vpc.main.id
  cidr_block     = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]
}

# Pattern 5: Map iteration
locals {
  instance_ami_map = {
    for name, config in var.instance_configs :
    name => config.ami
  }
}

resource "aws_instance" "servers" {
  for_each = var.instance_configs

  ami           = each.value.ami
  instance_type = each.value.type

  tags = {
    Name = each.key
  }
}
```

### Syntax Rules Summary

| Rule | Description | Example |
|-------|-------------|---------|
| **Block names** | Must be quoted strings | `resource "aws_instance" "web"` |
| **Strings** | Must use double quotes | `name = "server"` |
| **Comments** | `#` single-line, `/* */` multi-line | `# comment` |
| **Heredoc** | `<<-TAG ... TAG` | `script = <<-EOT` |
| **Lists** | Square brackets | `[1, 2, 3]` |
| **Maps** | Curly braces with `=>` | `{key => value}` |
| **Booleans** | `true` or `false` (no quotes) | `enabled = true` |
| **File extension** | `.tf` for HCL, `.tf.json` for JSON | `main.tf` |

## Interview Questions

**Q: What is the basic structure of an HCL file?**
**A:** HCL files contain blocks with optional block names, arguments (key-value pairs), and nested blocks. Structure: `block_type "name" { argument = value }`. All `.tf` files in directory are automatically loaded and merged. Common files: `main.tf`, `variables.tf`, `outputs.tf`.

**Q: How do you create comments in HCL?**
**A:** Single-line comments start with `#`, multi-line comments use `/* */`. Comments can be at end of lines or on separate lines. Example: `# This is a comment` or `/* Multi-line comment */`. Comments are ignored by Terraform.

**Q: What are the main data types in HCL?**
**A:** Primitive types: `string` (double-quoted), `number` (integers and floats), `bool` (true/false). Complex types: `list(...)` (ordered collection), `map(...)` (key-value pairs), `set(...)` (unique values), `object({...})` (named attributes), `tuple([...])` (fixed-length typed list).

**Q: What is a heredoc in HCL and when would you use it?**
**A:** Heredoc is a way to define multi-line strings without escaping. Syntax: `<<-TAG ... TAG`. Use `<<-` to strip leading whitespace. Used for scripts, configuration files, user_data. Example: `user_data = <<-EOT ... EOT`.

**Q: How do you reference variables in HCL expressions?**
**A:** Use `var.` prefix: `var.variable_name`. Example: `variable "region"`, reference as `var.region`. Works in any expression. For string interpolation: `"${var.region}"` or HCL 2: `"Region: ${var.region}"`.

**Q: What's the difference between list and set in HCL?**
**A:** List is ordered collection that allows duplicates. Set is unordered unique collection (duplicates removed automatically). Use list when order matters or duplicates allowed. Use set when order doesn't matter and uniqueness is important (e.g., security groups).

**Q: How do you create conditional expressions in HCL?**
**A:** Use ternary operator: `condition ? true_value : false_value`. Example: `var.environment == "prod" ? "t3.large" : "t3.micro"`. Can nest for complex logic. Use boolean expressions with `&&` (and), `||` (or), `!` (not).

**Q: What is the difference between `resource "type" "name"` and `data "type" "name"`?**
**A:** `resource` creates new infrastructure resources. `data` queries existing resources without creating them. Resources are managed and tracked in state. Data sources are read-only references to existing resources outside Terraform's control.

**Q: Can you mix HCL and JSON files in the same Terraform project?**
**A:** Yes, Terraform accepts both `.tf` (HCL) and `.tf.json` (JSON) files. All files in directory are merged into configuration. However, mixing is uncommon - typically use one format. JSON is used when generating config programmatically.

**Q: How do you organize Terraform configuration files?**
**A:** Common patterns: `main.tf` (resources, providers), `variables.tf` (input variables), `outputs.tf` (output values), `provider.tf` (provider config), `backend.tf` (state backend). File separation is optional - all can be in one file. Choose organization that fits your team's preferences.

**Q: What does the `<<-` prefix mean in heredoc?**
**A:** The `-` in `<<-TAG` removes leading whitespace from each line in heredoc content. Use `<<-` for indented heredocs to keep configuration readable. Without `-`, leading whitespace is preserved. Compare: `<<-EOT` (strips whitespace) vs `<<EOT` (preserves whitespace).

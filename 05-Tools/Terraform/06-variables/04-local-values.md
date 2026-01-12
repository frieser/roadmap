---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Local Values

## Summary

**Local values** (also called locals) in Terraform are temporary, computed values used within a single configuration. Unlike input variables, locals are not parameterized and can only be referenced within the same configuration where they're defined. Locals enable code reuse, simplify expressions, and avoid repetition without exposing anything outside the config.

## Detailed Explanation

### Locals vs Variables

| Aspect | Input Variables | Local Values |
|---------|----------------|------------|
| **Purpose** | Parameterize configuration | Internal code reuse |
| **Defined** | `variable` block | `locals` block |
| **Scope** | Global (entire config) | Local (within block) |
| **Parameterized** | Yes, passed via CLI/env/.tfvars | No, only computed |
| **State** | Stored in state file | Not persisted to state |
| **Access** | From anywhere (outputs, other modules) | Only within same config |
| **Use Case** | Infrastructure customization | Code simplification |

### Local Syntax

```hcl
# Locals block definition
locals {
  # Single value
  instance_name = "web-server"

  # Map of values
  instance_sizes = {
    small  = "t3.micro"
    medium = "t3.small"
    large  = "t3.large"
  }

  # List of values
  availability_zones = [
    "us-east-1a",
    "us-east-1b",
    "us-east-1c"
  ]
}
```

### Local Value Types

```hcl
# String
locals {
  app_name = "my-application"
  environment_name = "${var.project}-${var.environment}"
}

# Number
locals {
  instance_count = 3
  instance_port = 8080
}

# Boolean
locals {
  enable_monitoring = var.environment == "prod"
}

# List
locals {
  default_tags = [
    {
      Name        = "managed-by-terraform"
      Source      = "https://github.com/org/terraform-modules"
    },
    {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Owner       = "team-platform"
    }
  ]
}

# Map
locals {
  instance_types = {
    dev  = "t3.micro"
    staging = "t3.small"
    prod  = "t3.large"
  }
}

# Object
locals {
  database_config = {
    engine     = "postgres"
    version    = "14"
    port       = 5432
  }
}
```

### Using Locals in Resources

```hcl
# Use local to compute values
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "team-platform"
  }

  instance_config = {
    ami           = var.instance_ami
    instance_type = var.instance_types[var.environment]
  }
}

resource "aws_instance" "web" {
  ami           = local.instance_config.ami
  instance_type = local.instance_config.instance_type

  tags = merge(local.common_tags, {
    Name = "${local.app_name}-${local.environment_name}"
  })
}
```

### Local Expressions and Functions

```hcl
# String manipulation
locals {
  # Concatenation
  full_name = "${var.project}-${var.environment}"
  
  # String functions
  upper_name = upper(var.project_name)
  lower_name = lower(var.project_name)
  
  # Substring
  subnet_name = substr(local.subnet_name, 0, 8)
}

# Mathematical operations
locals {
  total_instances = var.web_count + var.db_count
  web_instances  = var.web_count
  db_instances  = var.db_count
}

# Conditional logic
locals {
  is_prod = var.environment == "prod"
  should_backup = var.environment == "prod" || var.environment == "staging"
}
```

### Map and List Operations

```hcl
# Map iteration
locals {
  subnet_cidrs = {
    for name, cidr in var.vpc_cidr_map : cidr => cidr
  }
}

# List comprehension
locals {
  public_subnet_ids = [
    for subnet in var.subnets :
      subnet.id if !contains(local.private_subnet_ids, subnet.id)
  ]
}

# Map filtering
locals {
  production_subnets = [
    for subnet in var.subnets :
    subnet.cidr if strcontains(subnet.cidr, "prod-")
  ]
}
```

### Nested Locals

```hcl
# Module input values (computed locally)
locals {
  # Module outputs
  vpc_id     = module.vpc.vpc_id
  subnet_ids  = module.vpc.subnet_ids

  # Computed configuration
  security_group_ids = concat(
    module.security_groups.web_sg_id,
    module.security_groups.db_sg_id
  )

  # Application configuration
  app_endpoint = "${module.app.endpoint}:${var.app_port}"
}

  db_connection_string = "postgres://${module.db.username}:${var.db_password}@${module.db.endpoint}:${module.db.port}"
}
```

### Locals with Data Sources

```hcl
# Query external resources
data "aws_vpc" "existing" {
  id = "vpc-1234567890"

  data "aws_subnet" "database" {
  filter {
    name   = "tag:Database"
    values = ["true"]
  }

  vpc_id = data.aws_vpc.existing.id
}

# Local value based on data source
locals {
  database_subnet_id = data.aws_subnet.database.id
}
```

### Locals vs Outputs

| Aspect | Locals | Outputs |
|---------|--------|---------|
| **Scope** | Within configuration | Across configurations |
| **State Persistence** | Not stored | Stored in state |
| **Re-evaluates** | Every apply | Computed once per plan |
| **Purpose** | Internal reuse | External communication |
| **Change Behavior** | Always recomputed | Shows only when changed | Can't use `-refresh-only` |

### Common Local Patterns

```hcl
# Pattern 1: Common tags
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "team-platform"
    CostCenter  = "engineering"
  }
}

resource "aws_instance" "web" {
  tags = merge(local.common_tags, {
    Name = "web-server"
  })
}

# Pattern 2: Conditional logic
locals {
  instance_type = local.instance_types[var.environment]
  monitoring   = local.is_prod ? true : false
}

# Pattern 3: Computed values
locals {
  # CIDR subnet calculation
  web_subnet_cidr = cidrsubnet(local.vpc_cidr, 8, 0)
  db_subnet_cidr  = cidrsubnet(local.vpc_cidr, 8, 1)

  # Resource naming
  web_instance_name = "web-server"
  db_instance_name = "database-server"
}
```

### Locals in Modules

```hcl
# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}

# root main.tf
locals {
  vpc_id = module.vpc.vpc_id
}

resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
}
}
```

### Best Practices

```hcl
# ✅ DO: Use locals for repeated expressions
locals {
  # Bad: var.project_name repeated everywhere
  # Good: local.project_name defined once
  common_tags = merge(var.tags, { ... })
}

# ✅ DO: Group related locals
locals {
  # Good: network values, compute values, configuration
}

# ❌ DON'T: Overuse locals
locals {
  # Bad: locals.x, locals.y, locals.z for everything
}

# ✅ DO: Use descriptive names
locals {
  # Good: instance_config, subnet_config
}

# ❌ DON'T: Make everything a local
# Keep truly reusable values as locals, constants as defaults
```

## Interview Questions

**Q: What are Terraform locals and when should you use them?**
**A:** Locals (defined in `locals` block) are temporary, computed values used within configuration. Use them for: 1) Code reuse (avoid repetition), 2) Simplifying complex expressions (compute once, use many times), 3) Grouping related values together, 4) Conditional logic (if/else expressions), 5) Constants (fixed values that don't change).

**Q: What's the difference between locals and input variables?**
**A:** Input variables are parameterized (can be passed via CLI/env/.tfvars), global scope, persisted in state. Locals are not parameterized, only computed within same config, not persisted. Input variables are for external customization; locals are for internal simplification.

**Q: Can locals reference outputs from the same configuration?**
**A:** Yes, locals can reference outputs just like resources. Example: `vpc_id = module.vpc.vpc_id`. Locals are evaluated at plan time, outputs are created during apply. Both are available in same configuration context.

**Q: What happens to locals during `terraform plan` vs `terraform apply`?**
**A:** During `terraform plan`, locals are evaluated to determine planned changes. During `terraform apply`, locals are also evaluated but resources are created/updated. Locals never change during apply (they're not resources). Locals are recomputed for each apply but not stored persistently.

**Q: Can locals reference data sources?**
**A:** Yes, locals can reference data source attributes just like resources. Example: `vpc_id = data.aws_vpc.existing.id`. This enables computed values based on external infrastructure queries or lookups.

**Q: How do you organize locals in large Terraform configurations?**
**A:** Organize locals logically: 1) Common values at top (tags, naming conventions), 2) Group by function (network locals, compute locals, config locals), 3) Module-related locals (vpc_id, subnet_ids), 4) Conditional values (environment-specific logic). This improves readability and maintenance.

**Q: Can locals use expressions to simplify configuration?**
**A:** Yes, compute complex values once in locals, reference them multiple times. Example: `locals { subnet_cidrs = { for name, cidr in var.vpc_cidr_map : cidr }` then reference `local.subnet_cidrs.web` instead of recomputing. This reduces errors and improves performance.

**Q: What's the difference between locals and `terraform` and `tainted` resources?**
**A:** Locals are computed values, not infrastructure resources. Taint is a metadata flag on resources indicating issues. Locals don't become tainted. Locals are evaluated fresh each apply; tainted resources retain their taint until explicitly untainted.

**Q: How do you use locals with `for_each` and `count`?**
**A:** Can use `for_each` and `count` in locals to iterate over data collections. Example: `locals { instance_ips = aws_instance.servers[*].public_ip }` or `locals { subnet_names = aws_subnet.public[*].id }`. Locals are computed at plan time, not at apply time.

**Q: Can you nest locals inside modules or resource blocks?**
**A:** No, locals can only be defined at configuration top level or within `data` and `locals` blocks. Cannot nest locals inside `resource` or `module` blocks. Use locals at module level for values shared by multiple resources in that module.

**Q: How do locals affect Terraform state file?**
**A:** Locals themselves don't create state entries. Only resources create state. However, locals influence plan output (shows computed values). Terraform stores only resource state, not local values. Locals are recomputed but not persisted across runs.

**Q: When should you use locals instead of variables?**
**A:** Use locals for: 1) Computed values used multiple times (avoid repetition), 2) Complex expressions (conditional logic, map lookups), 3) Constants (fixed values), 4) Grouping related computed values. Use variables only when you need to externalize customization (inputs).

---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# depends_on Meta-Argument

## Summary

**`depends_on`** is a Terraform meta-argument that explicitly creates resource dependencies, controlling the order of resource creation and updates. While Terraform automatically infers dependencies from resource references, `depends_on` is useful for hidden dependencies or when you need specific execution order.

## Detailed Explanation

### Dependency Inference

```hcl
# Automatic dependencies (inferred from references)
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Automatic dependency
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id  # Automatic dependency
  ami       = "ami-..."
}
```

### Explicit depends_on

```hcl
# Hidden dependency (no reference in resource)
resource "aws_nat_gateway" "main" {
  subnet_id = aws_subnet.public.id
}

resource "aws_instance" "web" {
  ami       = "ami-..."
  subnet_id = aws_subnet.public.id

  # Explicit: Instance depends on NAT gateway
  # (No direct reference to NAT gateway)
  depends_on = [aws_nat_gateway.main]
}
```

### depends_on Syntax

```hcl
# List of resources
depends_on = [aws_vpc.main, aws_security_group.web]

# Module dependencies
depends_on = [module.vpc, module.database]

# Combined: resources and modules
depends_on = [aws_vpc.main, module.networking]
```

### Use Cases

```hcl
# Use case 1: Hidden infrastructure dependency
resource "aws_iam_role" "ecs_task_role" {
  assume_role_policy = data.aws_iam_policy_document.ecs.json
}

resource "aws_iam_role_policy_attachment" "attach" {
  role       = aws_iam_role.ecs_task_role.name
  policy_arn = aws_iam_policy.policy.arn

  # Explicit: Attachment depends on role creation
  depends_on = [aws_iam_role.ecs_task_role]
}

# Use case 2: Resource created outside Terraform
resource "aws_instance" "web" {
  # Depends on existing S3 bucket (not managed by Terraform)
  # Bucket must exist before instance can be created
  depends_on = [data.aws_s3_bucket.existing]
}

# Use case 3: Specific creation order
resource "aws_subnet" "subnet_1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_subnet" "subnet_2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"
}

resource "aws_subnet" "subnet_3" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.3.0/24"
}

resource "aws_instance" "web" {
  # Ensure specific subnet order
  depends_on = [
    aws_subnet.subnet_1,
    aws_subnet.subnet_2,
    aws_subnet.subnet_3
  ]
  subnet_id = aws_subnet.subnet_1.id
}
```

### Module depends_on

```hcl
# Module depends on another module
module "database" {
  source = "./modules/rds"
}

module "application" {
  source     = "./modules/app"
  depends_on = [module.database]

  # App depends on database being ready
  database_endpoint = module.database.endpoint
}
```

### Multiple dependencies

```hcl
# Multiple resources in depends_on
resource "aws_instance" "web" {
  ami       = "ami-..."
  subnet_id = aws_subnet.public.id

  # Depends on multiple resources
  depends_on = [
    aws_vpc.main,
    aws_internet_gateway.gw,
    aws_route_table.public
  ]
}
```

### Circular Dependencies

```hcl
# ❌ CIRCULAR DEPENDENCY (invalid)
resource "aws_instance" "web" {
  depends_on = [aws_instance.database]  # Web depends on DB
}

resource "aws_instance" "database" {
  depends_on = [aws_instance.web]  # DB depends on web
}

# Error: Cycle: aws_instance.web -> aws_instance.database -> aws_instance.web
# Solution: Remove circular dependency, redesign
```

### depends_on vs Implicit Dependencies

| Aspect | Implicit | Explicit (depends_on) |
|---------|---------|-----------------------|
| **Trigger** | Resource reference in code | Explicit declaration |
| **Visibility** | Visible in configuration | Hidden (not obvious) |
| **Automatic** | Always inferred when referenced | Must be manually specified |
| **Use Case** | Data flow between resources | External requirements, order control |

### Conditional depends_on

```hcl
# Conditional dependency using count
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0

  depends_on = [
    # Only depend on NAT gateway if creating instance
    var.create_instance ? aws_nat_gateway.main : null
  ]

  # Filter out null values using compact
  depends_on = compact([
    var.create_instance ? aws_nat_gateway.main : null
  ])
}
```

### Best Practices

```hcl
# ✅ DO: Use implicit dependencies when possible
resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id  # Clear, automatic
}

# ❌ DON'T: Use depends_on when reference works
resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id
  depends_on = [aws_subnet.public]  # Redundant!
}

# ✅ DO: Use depends_on for hidden dependencies
resource "aws_iam_instance_profile" "app_profile" {
  # Depends on role created by separate module
  depends_on = [aws_iam_role.app_role]
}

# ❌ DON'T: Use depends_on to create circular deps
# Break circular dependencies instead

# ✅ DO: Document why depends_on is needed
resource "aws_instance" "web" {
  depends_on = [aws_nat_gateway.main]

  # Comment explaining dependency
  # Instance requires NAT gateway for internet access
}
```

### Complete Example

```hcl
# main.tf
variable "create_instance" {
  type    = bool
  default = false
}

# VPC infrastructure
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "internet_access" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  # Subnet depends on route (route must exist before subnet)
  depends_on = [aws_route.internet_access]
}

resource "aws_eip" "nat" {
  depends_on = [aws_route.internet_access]
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.allocation_id
  subnet_id     = aws_subnet.public.id
}

# NAT Gateway
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id

  # Explicit: Instance depends on NAT gateway
  depends_on = [
    aws_vpc.main,
    aws_internet_gateway.igw,
    aws_route_table.public,
    aws_route.internet_access,
    aws_nat_gateway.nat
  ]
}

output "instance_id" {
  value = var.create_instance ? aws_instance.web[0].id : null
}
```

### Dependency Graph

```bash
# View dependency graph
terraform graph

# Output:
# digraph {
#   [root] aws_vpc.main;
#   [root] aws_internet_gateway.igw -> aws_vpc.main;
#   [root] aws_route.internet_access -> aws_vpc.main;
#   [root] aws_nat_gateway.nat -> aws_vpc.main;
#   [root] aws_subnet.public -> aws_route.internet_access;
#   [root] aws_instance.web -> aws_nat_gateway.nat;
# }

# Visualize with Graphviz
terraform graph | dot -Tpng > dependency-graph.png
```

### Troubleshooting

```bash
# Problem: Resources created in wrong order
# Solution: Use depends_on to enforce order

# Problem: Plan fails with circular dependency
# Error: Cycle: resource.a -> resource.b -> resource.a
# Solution: Redesign to break cycle

# Problem: depends_on not working
# Solution: Check resource names match exactly

# Problem: Module depends_on not finding resources
# Solution: Use module.<name>.resource format
```

## Interview Questions

**Q: What is `depends_on` meta-argument in Terraform?**
**A:** `depends_on` explicitly creates resource dependencies, controlling execution order. Syntax: `depends_on = [resource.type.name, ...]`. Used for hidden dependencies or when specific creation order is required. Terraform also infers automatic dependencies from resource references.

**Q: When should you use `depends_on` versus automatic dependency inference?**
**A:** Use automatic dependency (resource references) when possible - clearer and less error-prone. Use `depends_on` for: 1) Hidden dependencies not obvious from code, 2) Resources created outside Terraform, 3) Specific execution order requirements, 4) Breaking circular dependencies.

**Q: What happens if you create a circular dependency with `depends_on`?**
**A:** Terraform detects circular dependencies and fails with error: `Cycle: resource.a -> resource.b -> resource.a`. Plan stage will fail. Solution: redesign configuration to break cycle, remove one of the dependency links, or restructure resources.

**Q: How do you use `depends_on` with Terraform modules?**
**A:** For module dependencies: `module "app" { depends_on = [module.database] }`. For resources within modules: child module uses depends_on normally. Access module resources: `depends_on = [module.vpc.aws_vpc.main]`. Creates ordering between module outputs and consuming resources.

**Q: Can you use `depends_on` with data sources?**
**A:** Yes, `depends_on` works with data sources and resources. Example: `resource "aws_instance" "web" { depends_on = [data.aws_s3_bucket.config] }`. Ensures data source queries complete before resource creation. Useful when data requires other data.

**Q: How does Terraform's dependency graph work with `depends_on`?**
**A:** Terraform builds a directed acyclic graph (DAG) of all dependencies. Each node is a resource, edges are dependencies (automatic and `depends_on`). Terraform performs topological sort to determine creation order. Must be acyclic (no circular dependencies) for successful plan.

**Q: What are common mistakes when using `depends_on`?**
**A:** Common mistakes: 1) Redundant dependencies (using `depends_on` when resource reference already creates dependency), 2) Creating circular dependencies, 3) Incorrect resource names in depends_on list, 4) Overusing depends_on when automatic would work, 5) Forgetting dependencies when order matters.

**Q: How do you view the Terraform dependency graph?**
**A:** Use `terraform graph` command to generate Graphviz DOT output. Visualize with `terraform graph | dot -Tpng > graph.png`. Shows all resources and dependencies as directed edges. Useful for understanding complex infrastructure relationships.

**Q: Can you use conditional logic with `depends_on`?**
**A:** No direct conditional syntax, but can use `compact()` to filter null values: `depends_on = compact([var.condition ? resource.a : null])`. This creates conditional dependencies based on variable values. Only non-null dependencies are included in the list.

**Q: How does `depends_on` affect `terraform plan` output?**
**A:** `depends_on` ensures resources in depends_on list are created before the depending resource. Plan output shows dependency order but doesn't change what resources are created (that's determined by configuration). Useful for verifying Terraform will create in expected order.

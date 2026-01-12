---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# Secret Management

## Summary

**Secret management** in Terraform involves handling sensitive data (passwords, API keys, certificates) securely. Never store secrets in configuration files, state, or outputs. Use environment variables, secret managers (AWS Secrets Manager, HashiCorp Vault), or parameter stores (AWS SSM Parameter Store, Azure Key Vault). Terraform provides `sensitive` flag to hide sensitive values from logs and outputs.

## Detailed Explanation

### The Problem with Hard-Coded Secrets

```hcl
# ❌ NEVER DO THIS: Hard-code secrets in Terraform
resource "aws_instance" "database" {
  password = "my-secret-password"  # SECURITY RISK!
}

resource "aws_rds_cluster" "main" {
  master_username = "admin"  # SECURITY RISK!
  master_password = "SuperSecret123!"  # SECURITY RISK!
}

# State file will contain unencrypted secrets
# Anyone with state access can read passwords
# Git history contains secrets if state committed
```

### Sensitive Variable Flag

```hcl
# Mark variable as sensitive
variable "database_password" {
  type      = string
  sensitive = true  # Hides from logs/outputs
  description = "Database password (never hard-code!)"
}

# Usage
resource "aws_db_instance" "main" {
  password = var.database_password

  # Output (also marked sensitive)
  lifecycle {
    ignore_changes = [password]  # Ignore in plan output
  }
}

output "db_connection_string" {
  sensitive = true
  value     = "postgres://user:${var.database_password}@db:5432/dbname"
}
```

### Environment Variables (CI/CD Friendly)

```bash
# Option 1: Direct environment variables
export TF_VAR_database_password=$(vault kv get -field=password secret/database)
terraform plan

# Option 2: Terraform variable prefix
export TF_VAR_database_password="secure-value"
terraform apply -var="database_password=$TF_VAR_database_password"

# Option 3: TF_VAR_ prefix (auto-loaded)
export TF_VAR_database_password  # Terraform auto-loads
# Use without TF_VAR_ prefix in terraform code
```

### AWS Secrets Manager

```hcl
# Use Terraform provider data source
data "aws_secretsmanager_secret" "database_password" {
  name = "prod/database/password"

  # Requires IAM permissions
  # secretsmanager:GetSecretValue
}

resource "aws_db_instance" "main" {
  allocated_storage     = 20
  storage_type        = "gp2"
  engine               = "postgres"
  master_username     = "dbadmin"
  master_password     = data.aws_secretsmanager_secret.database_password.secret_string
}
```

### AWS SSM Parameter Store

```hcl
# Store parameter before Terraform
# aws ssm put-parameter \
#   --name "/myapp/database/password" \
#   --value "secure-password" \
#   --type "SecureString" \
#   --description "Database password"

# Retrieve parameter in Terraform
data "aws_ssm_parameter" "database_password" {
  name = "/myapp/database/password"
  with_decryption = true
}

resource "aws_db_instance" "main" {
  master_username     = "dbadmin"
  master_password     = data.aws_ssm_parameter.database_password.value
}
```

### Azure Key Vault

```hcl
# Azure Key Vault data source
data "azurerm_key_vault_secret" "database_password" {
  name         = "database-password"
  key_vault_id = azurerm_key_vault.this.id
}

resource "azurerm_mssql_server" "main" {
  administrator_login = "sqladmin"
  administrator_password = data.azurerm_key_vault_secret.database_password.value
}
```

### HashiCorp Vault

```bash
# Retrieve secret from Vault
vault kv get -field=password secret/database

# Set as environment variable
export TF_VAR_database_password=$(vault kv get -field=password secret/database)
```

```hcl
# Terraform doesn't have native Vault provider
# Use environment variables to pass secrets
variable "database_password" {
  type      = string
  sensitive = true
}

resource "aws_instance" "web" {
  user_data = <<-EOT
    #!/bin/bash
    export DB_PASSWORD="${TF_VAR_database_password}"
    echo "Password set from Vault"
  EOT
}
```

### Terraform Cloud Workspaces Variables

```hcl
# Set sensitive variables in workspace (Terraform Cloud)
# Via UI or API
# Not visible in configuration

variable "api_key" {
  type      = string
  sensitive = true
}

resource "aws_api_gateway" "main" {
  # Uses workspace variable
  # No secrets in state file
}
```

### Provider Configuration

```hcl
# Configure provider credentials (not secrets)
provider "aws" {
  region = "us-west-2"
  # Credentials from:
  # - AWS environment variables
  # - ~/.aws/credentials file
  # - EC2 instance profile (if running on EC2)
  # - No hard-coded access keys!
}

provider "google" {
  project = "my-project"
  # Credentials from:
  # - GOOGLE_APPLICATION_CREDENTIALS
  # - ADC file
  # - GKE authentication
}

provider "azurerm" {
  features {}
  # Credentials from:
  # - service principal + client secret
  # - Azure CLI authentication
  # - Managed identity
}
```

### Provisioners and Secrets

```hcl
# ❌ DON'T: Pass secrets to provisioners
resource "aws_instance" "web" {
  ami           = "ami-..."
  instance_type = "t3.micro"

  # Pass via environment variable
  provisioner "remote-exec" {
    inline = [
      "export DB_PASSWORD=${var.db_password}",
      "mysqladmin -p${var.db_password}"
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
}
```

### Outputting Sensitive Data

```hcl
# Sensitive outputs (not displayed in CLI output)
output "database_endpoint" {
  description = "Database connection string (sensitive)"
  sensitive = true
  value     = "postgres://db.internal:5432/mydb"
}

output "api_key" {
  description = "API key (sensitive)"
  sensitive = true
  value     = var.api_key
}

# Non-sensitive output
output "instance_id" {
  description = "Instance ID"
  value       = aws_instance.web.id
}
```

### Best Practices

| Practice | Description | Example |
|-----------|-------------|---------|
| **Never hard-code** | Don't store secrets in `.tf` files | `password = "secret123"` is forbidden |
| **Use sensitive flag** | Mark secrets with `sensitive = true` | Prevents logging and CLI display |
| **Use secrets manager** | AWS Secrets Manager, SSM, Vault | Centralized secret management |
| **Environment variables** | CI/CD friendly secret passing | `export TF_VAR_secret=value` |
| **Rotate secrets** | Regularly rotate passwords, keys, certificates | Automate rotation where possible |
| **Encrypt state** | Enable S3/KMS encryption | Protects state at rest |
| **Avoid outputs** | Don't output sensitive values | Use secrets manager instead |
| **Least privilege** | IAM policies for minimal access | Only necessary read/write permissions |
| **Audit trail** | Log secret access without logging values | Track who accessed what when |

### Complete Example

```hcl
# variables.tf
variable "environment" {
  type    = string
  default = "prod"
}

variable "db_instance_class" {
  type    = string
  default = "db.t3.micro"
}

# main.tf
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

# Retrieve secrets from AWS Secrets Manager
data "aws_secretsmanager_secret" "db_password" {
  name = "prod/${var.environment}/db/password"
}

data "aws_secretsmanager_secret" "db_host" {
  name = "prod/${var.environment}/db/host"
}

resource "aws_db_instance" "main" {
  identifier        = "database"
  engine            = "postgres"
  instance_class     = var.db_instance_class
  allocated_storage  = 20
  storage_type      = "gp2"

  master_username = "dbadmin"
  master_password = data.aws_secretsmanager_secret.db_password.secret_string

  # Pass database endpoint via output
  lifecycle {
    ignore_changes = [master_password]  # Don't log password
  }
}

output "db_endpoint" {
  description = "Database endpoint (retrieve from Secrets Manager)"
  sensitive   = true
  value       = aws_db_instance.main.endpoint
}

output "db_host" {
  description = "Database host"
  sensitive   = true
  value       = data.aws_secretsmanager_secret.db_host.secret_string
}

output "db_port" {
  description = "Database port"
  sensitive   = false  # Not secret
  value       = aws_db_instance.main.port
}
```

```bash
# CI/CD setup
# .github/workflows/deploy.yml
name: Deploy Database

on:
  workflow_dispatch:

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Store secret in AWS Secrets Manager
        run: |
          DB_PASSWORD=$(openssl rand -base64 32)
          aws secretsmanager put-secret-value \
            --secret-id "prod/db/password" \
            --secret-string "$DB_PASSWORD" \
            --description "Production database password"

      - name: Terraform Apply
        run: |
          # No secrets in Terraform code
          # Terraform retrieves from Secrets Manager
          terraform apply -auto-approve
```

### Common Mistakes

```hcl
# ❌ MISTAKE 1: Secrets in variables.tf
variable "password" {
  default = "secret123"  # Visible in version control!
}

# ❌ MISTAKE 2: Secrets in outputs.tf
output "api_key" {
  value = var.api_key  # Visible in logs!
}

# ❌ MISTAKE 3: Secrets in state file
# terraform.tfstate contains unencrypted secrets
# Anyone with state bucket access can read

# ❌ MISTAKE 4: Secrets in user_data
resource "aws_instance" "web" {
  user_data = "password=secret123"  # Logged to console!
}

# ❌ MISTAKE 5: No sensitive flag on outputs
output "password" {
  value = var.password  # Displayed in CLI and logs!
}
```

### Secret Rotation Strategy

```bash
# Automated rotation with AWS Secrets Manager
# 1. Generate new password
NEW_PASSWORD=$(openssl rand -base64 32 | base64 -d)

# 2. Update secret in Secrets Manager
aws secretsmanager put-secret-value \
  --secret-id "prod/db/password" \
  --secret-string "$NEW_PASSWORD" \
  --description "Rotated password"

# 3. Rotate in application
# Application reads new secret on next connection
# Implement logic to gracefully re-authenticate
```

## Interview Questions

**Q: How do you handle secrets in Terraform securely?**
**A:** Never hard-code secrets in `.tf` files. Use: 1) Environment variables (`TF_VAR_name=value`), 2) Secret managers (AWS Secrets Manager, SSM Parameter Store, HashiCorp Vault), 3) Terraform Cloud workspace variables, 4) CI/CD secret injection. Mark variables and outputs as `sensitive = true` to prevent logging. Retrieve secrets at runtime.

**Q: What does the `sensitive = true` flag do in Terraform?**
**A:** `sensitive = true` marks values as sensitive, preventing them from being displayed in CLI output (`terraform output`) and logs. State file still contains encrypted value, but CLI masks it. Use for passwords, API keys, connection strings, or any sensitive configuration data.

**Q: How do you use AWS Secrets Manager with Terraform?**
**A:** Use `data "aws_secretsmanager_secret"` data source to retrieve secrets by name. Pass to resource configurations: `master_password = data.aws_secretsmanager_secret.db_password.secret_string`. Requires IAM permissions: `secretsmanager:GetSecretValue`. Secret must exist before Terraform run. Use for rotating secrets or storing application-specific credentials.

**Q: How do you pass secrets from CI/CD to Terraform?**
**A:** Methods: 1) Environment variables: `TF_VAR_secret=value`, 2) Secret injection from GitHub Actions/AWS Secrets Manager, 3) Terraform Cloud workspace variables (configured in UI), 4) Use secret provider that integrates with your secret manager. Never commit secrets to repository or print in CI/CD logs.

**Q: What's the problem with storing secrets in Terraform state files?**
**A:** State files stored in remote backends (S3, GCS) might be encrypted, but: 1) State file can be downloaded, 2) State history contains all past secrets, 3) Local state files are unencrypted, 4) Anyone with state access can read secrets. Risk of credential exposure through version control or shared storage.

**Q: How do you output sensitive values from Terraform without exposing them?**
**A:** Use `output "name" { sensitive = true }` to mask from CLI output. For programmatic access, use `terraform output -json` and parse with jq, then access sensitive values. Terraform CLI still displays `(sensitive)` marker instead of value. Best practice: don't output secrets at all if possible.

**Q: How do you rotate secrets used by Terraform resources?**
**A:** Update secret in secret manager (AWS Secrets Manager, Vault) without changing Terraform code. Resource automatically uses new secret on next `terraform apply`. Some resources (RDS, ElastiCache) might require recreation for password changes. Use `lifecycle` blocks or provider-specific rotation mechanisms for managed databases.

**Q: What's the best practice for managing database passwords in Terraform?**
**A:** Best practice: Store passwords in secret manager (AWS Secrets Manager, RDS rotation), not in Terraform. Use `data "aws_secretsmanager_secret"` to retrieve. For RDS: Use `master_password = data.aws_secretsmanager_secret.db_password.secret_string`. Enable automatic rotation if supported. Mark password-related outputs as `sensitive = true`.

**Q: Can you use environment variables for all sensitive data in Terraform?**
**A:** Yes, for small environments or personal projects. Limitations: 1) CI/CD platforms limit environment variable length, 2) Not version controlled, 3) Security risk if repository compromised. For production: use secret managers or parameter stores with proper access controls. Environment variables better for development/testing.

**Q: How do you handle secrets in Terraform Cloud?**
**A:** Configure secrets via Terraform Cloud UI in workspace variables. Variables set in UI (not in code) are accessible to runs. Mark as sensitive in output definitions. Secrets never stored in state file. Terraform Cloud manages secret injection securely at runtime. Workspace isolation for different environments.

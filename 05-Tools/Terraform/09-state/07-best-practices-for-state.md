---
tags: ['terraform', 'iac', 'tools', 'roadmap']
---

# State Management Best Practices

## Summary

Effective state management is critical for Terraform success, especially in production environments. Best practices include: using remote backends with locking, enabling encryption, separating environments, implementing access controls, backing up state, avoiding manual state modification, and documenting state architecture. These practices prevent state corruption, unauthorized access, and collaboration conflicts.

## Detailed Explanation

### 1. Use Remote State Backends

```hcl
# ✅ DO: Always use remote backend in production
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key           = "prod/terraform.tfstate"
    region        = "us-west-2"
    encrypt       = true
    dynamodb_table = "terraform-locks"
  }
}

# ❌ DON'T: Use local state in teams
# Local state causes conflicts and no locking
```

### 2. Enable State Locking

```bash
# Automatic locking with S3 + DynamoDB
terraform {
  backend "s3" {
    dynamodb_table = "terraform-locks"
  }
}

# Manual locking (not recommended)
# Use file locks or coordination
```

### 3. Encrypt State at Rest

```hcl
# AWS S3 server-side encryption
terraform {
  backend "s3" {
    encrypt       = true              # SSE-S3 encryption
    # server_side_encryption_configuration = "aws:kms:..."  # Or KMS
    kms_key_id    = "arn:aws:kms:us-west-2:123456789012"
  }
}

# GCS encryption
terraform {
  backend "gcs" {
    encryption_key = "projects/my-project/locations/us/key1"
  }
}
```

### 4. Separate Environments

```bash
# Environment isolation strategy
# Option 1: Separate state files
prod/
├── terraform.tf (backend: prod/terraform.tfstate)
staging/
├── terraform.tf (backend: staging/terraform.tfstate)

# Option 2: Same bucket with prefixes
terraform-state-prod/prod/terraform.tfstate
terraform-state-prod/staging/terraform.tfstate
terraform-state-prod/dev/terraform.tfstate

# Option 3: Terraform Cloud workspaces
terraform workspace select prod
terraform workspace select staging
terraform workspace select dev
```

### 5. Secure State Access

```bash
# IAM policy for state bucket
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::terraform-state-prod",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/TerraformTeam"
      }
    }
  ]
}

# Use IAM roles instead of access keys in CI/CD
```

### 6. Enable State Versioning

```bash
# Enable S3 bucket versioning
aws s3api put-bucket-versioning \
  --bucket terra-state-prod \
  --status-enabled

# Terraform Cloud: Automatic
terraform {
  cloud {
    workspaces {
      name = "prod"
      # State versioned automatically
    }
  }
}
```

### 7. Never Modify State Manually

```bash
# ❌ DON'T: Edit terraform.tfstate directly
# Always use Terraform commands

# ✅ DO: Use terraform commands for state operations
terraform state list
terraform state show
terraform state pull

# ❌ DON'T: Commit state files to git
# State contains sensitive data, large binary

# ❌ DON'T: Share state files via email/slack
# Risk of unauthorized access
```

### 8. Back Up State Regularly

```bash
# S3 versioning provides automatic backups
# Access previous versions via AWS CLI
aws s3api list-object-versions \
  --bucket terra-state-prod \
  --key prod/terraform.tfstate

# Manual backup strategy
terraform state pull > backup-$(date +%Y%m%d).tfstate
```

### 9. Document State Architecture

```markdown
# STATE MANAGEMENT ARCHITECTURE

## Overview

Terraform state is stored remotely using S3 backend with DynamoDB locking.

## Backends

- **Development**: Local state (for rapid iteration)
- **Staging**: S3 backend (testing)
- **Production**: S3 backend (locked)

## Access Control

- **Read/Write**: Terraform team IAM role
- **Read-Only**: Security auditors IAM role
- **No Access**: All other accounts denied

## Workspaces

- `prod`: Production environment
- `staging`: Pre-production testing
- `dev`: Development environment

## Disaster Recovery

1. Enable S3 versioning on state bucket
2. Document restore procedure in runbook
3. Test restore in non-prod quarterly
4. Monitor Terraform Cloud state history
```

### 10. Monitor State File Size

```bash
# Check state file size
terraform show -json | jq '.resources | length' state.json
# Large state files slow down Terraform operations

# Strategies:
# - Split state across multiple state files
# - Remove old resources from state
# - Use workspaces for logical separation
```

### Complete Configuration Example

```hcl
# backend.tf
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    bucket         = "terraform-state-${var.environment}"
    key           = "${var.environment}/${var.project}/terraform.tfstate"
    region        = "us-west-2"
    encrypt       = true
    dynamodb_table = "terraform-locks-${var.environment}"
    kms_key_id    = var.kms_key_id
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

variable "environment" {
  type    = string
  default = "prod"
}

variable "project" {
  type    = string
  default = "main-platform"
}
```

```bash
# Bootstrap script
#!/bin/bash

ENV=${1:-dev}

terraform init \
  -backend-config=backend-${ENV}.tf \
  -reconfigure

echo "Terraform state initialized for $ENV environment"
```

### CI/CD State Management

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'

env:
  TF_VERSION: "1.6.0"
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        run: |
          aws configure set default.aws_access_key_id ${{ env.AWS_ACCESS_KEY_ID }}
          aws configure set default.aws_secret_access_key ${{ env.AWS_SECRET_ACCESS_KEY }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan -out=tfplan

  apply:
    runs-on: ubuntu-latest
    needs: plan
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        run: |
          aws configure set default.aws_access_key_id ${{ env.AWS_ACCESS_KEY_ID }}
          aws configure set default.aws_secret_access_key ${{ env.AWS_SECRET_ACCESS_KEY }}

      - name: Terraform Apply
        run: terraform apply tfplan
        env:
          TF_CLI_ARGS: "-no-color"
```

### Common Anti-Patterns

```hcl
# ❌ ANTI-PATTERN 1: Local state in production
# Problems: No locking, conflicts, security risk

# ❌ ANTI-PATTERN 2: Single state file for all environments
# Problems: Large state, slow operations, difficult to isolate

# ❌ ANTI-PATTERN 3: Commit state files to git
# Problems: Security risk, large binary in repo, history bloat

# ❌ ANTI-PATTERN 4: Manual state modification
# Problems: State corruption, desynchronization, unpredictable behavior

# ❌ ANTI-PATTERN 5: No encryption
# Problems: Sensitive data exposed in S3, compliance violations

# ❌ ANTI-PATTERN 6: No access control
# Problems: Any account with credentials can modify state, audit issues
```

### State Troubleshooting Guide

```bash
# Problem: State lock not releasing
terraform force-unlock <LOCK_ID>

# Problem: State corrupted
terraform state pull  # Pull remote state
# If corrupted, restore from S3 version

# Problem: Permission denied
# Check IAM policies for state bucket
# Verify credentials are correct

# Problem: Terraform cannot find state
# Verify backend configuration
# Check bucket name and key are correct
```

## Interview Questions

**Q: What are the most important best practices for Terraform state management?**
**A:** Critical practices: 1) Always use remote state backends in production, 2) Enable automatic state locking (S3+DynamoDB, Terraform Cloud), 3) Encrypt state at rest and in transit, 4) Separate environments (different state files or workspaces), 5) Restrict access with IAM policies, 6) Enable versioning/backups, 7) Never manually modify state files, 8) Document state architecture.

**Q: Why should you use remote state instead of local state?**
**A:** Remote state provides: automatic locking (prevents concurrent conflicts), team collaboration (shared access), encryption (data at rest), backups (version control), CI/CD integration (no local state), and security (no local files). Local state in teams leads to state corruption, difficult coordination, and security risks.

**Q: How do you implement state locking for Terraform?**
**A:** Terraform Cloud provides automatic locking. For S3 backends: use DynamoDB table for locking (`dynamodb_table = "terraform-locks"`). Terraform acquires lock in DynamoDB before operations, releases after. Consul backend also provides locking. Locking prevents concurrent operations that could corrupt state file.

**Q: What's the difference between S3 versioning and DynamoDB locking?**
**A:** S3 versioning stores historical versions of state file - enables rollback to previous states. DynamoDB backend stores only lock information (who holds lock, when acquired) in DynamoDB, not state data itself. S3 stores actual state file, DynamoDB provides locking mechanism. Used together: S3 for storage, DynamoDB for concurrency control.

**Q: How do you separate Terraform state across environments?**
**A:** Three approaches: 1) Different S3 buckets or keys (`prod/terraform.tfstate`, `dev/terraform.tfstate`), 2) Same bucket with prefixes (`terraform-state-prod/prod/`, `terraform-state-prod/dev/`), 3) Terraform Cloud workspaces (named environments). Choose based on team needs. Separate state prevents cross-environment interference and simplifies permissions.

**Q: How do you secure Terraform state from unauthorized access?**
**A:** Security measures: 1) Enable S3 bucket encryption (`encrypt = true`, KMS keys), 2) Block public access (`acl private`), 3) Use IAM policies for least privilege (read/write vs read-only), 4) Use IAM roles instead of access keys in automation, 5) Enable MFA for console access, 6) Rotate access keys regularly, 7) Monitor S3 access logs, 8) Never commit state files to git.

**Q: What happens if Terraform state file becomes corrupted?**
**A:** If S3 versioning enabled: Restore from previous version. If not: Terraform fails to read state. Recovery options: 1) Recreate infrastructure from code (destructive but clean), 2) Import existing resources (complex), 3) Restore from state backup if available. Prevention: enable versioning, regular backups, avoid manual state modification.

**Q: Why should you never manually edit Terraform state files?**
**A:** State files are binary JSON that Terraform manages. Manual editing causes: 1) State corruption (invalid JSON), 2) Desynchronization between code and actual infrastructure, 3) Unpredictable behavior, 4) Terraform errors. Always use Terraform CLI or API for state operations (`terraform state list`, `terraform state rm`, etc.).

**Q: How do you monitor Terraform state file size and performance?**
**A:** Monitor state file size with `terraform show -json | jq '.resources | length'`. Large state files slow down operations. Solutions: 1) Split state across multiple files by environment/team/module, 2) Remove old/deleted resources from state using `terraform state rm`, 3) Use `terraform refresh -target` instead of full refresh, 4) Enable workspaces for logical separation.

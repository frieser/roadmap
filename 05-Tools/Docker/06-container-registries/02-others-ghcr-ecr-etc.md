---
tags: ['docker', 'containers', 'registries', 'devops', 'tools', 'roadmap']
---

# Other Container Registries

## Summary

While Docker Hub is the most well-known container registry, many organizations use alternative registries for better security, performance, compliance, or integration with existing infrastructure. Popular alternatives include GitHub Container Registry (ghcr.io), AWS Elastic Container Registry (ECR), Google Artifact Registry (GAR), Azure Container Registry (ACR), and others. Each registry has unique features for authentication, caching, and access control.

## Detailed Explanation

### Why Use Alternative Registries

```yaml
# DITCH DOCKER HUB WHEN:
compliance_requirements:
  - Data must stay in specific region
  - Company policy prohibits public registries
  - Regulatory requirements (GDPR, HIPAA)

performance_needs:
  - Global content delivery needed
  - Private networking with registry
  - Build cache acceleration

integration_needs:
  - CI/CD platform integration
  - Existing cloud provider services
  - Single sign-on (SSO) requirements

# POPULAR ALTERNATIVES:
alternatives:
  github: "GitHub Container Registry (ghcr.io)"
  aws: "Elastic Container Registry (ECR)"
  google: "Google Artifact Registry (GAR)"
  azure: "Azure Container Registry (ACR)"
  gitlab: "GitLab Container Registry"
  quay: "Quay.io (Red Hat)"
  harbor: "Harbor (self-hosted)"
```

### GitHub Container Registry (ghcr.io)

```bash
# GITHUB CONTAINER REGISTRY

# Authenticate (requires GitHub PAT)
export CR_PAT=ghp_xxxxxxxxxxxx
echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin

# Image naming
ghcr.io/owner/image:tag
ghcr.io/owner/image:latest
ghcr.io/owner/image:v1.2.3

# GitHub Actions authentication (automatic)
# No manual auth needed in GitHub Actions
# Uses GITHUB_TOKEN automatically

# Push image
docker tag myapp:latest ghcr.io/myorg/myapp:latest
docker push ghcr.io/myorg/myapp:latest

# Pull image
docker pull ghcr.io/myorg/myapp:latest

# Visibility settings
# Public: Anyone can pull
# Private: Requires authentication
# Set in repository settings → Packages → Container image

# GitHub Packages UI
# View: https://github.com/ORG/REPO/pkages/container/myapp
# Manage: Delete images, update visibility, view usage
```

```yaml
# GitHub Actions example
name: Build and Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

### AWS Elastic Container Registry (ECR)

```bash
# AWS ECR AUTHENTICATION

# Install AWS CLI
# Credentials via ~/.aws/credentials or environment

# Configure ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Image naming
<account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>:tag
123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Create repository
aws ecr create-repository \
  --repository-name myapp \
  --region us-east-1

# Push image
docker tag myapp:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Pull image
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Lifecycle policies (auto-delete old images)
aws ecr put-lifecycle-policy \
  --repository-name myapp \
  --policy-text 'file://policy.json'

# policy.json example:
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images",
      "selection": {
        "tagStatus": "tagged",
        "tagPrefixList": ["v"],
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

```yaml
# AWS CDK / Terraform ECR setup
resource "aws_ecr_repository" "myapp" {
  name                 = "myapp"
  image_tag_mutability = "MUTABLE"

  lifecycle_policy {
    rules {
      rule_priority = 1
      selection {
        tag_status   = "tagged"
        tag_prefix_list = ["v"]
        count_type  = "imageCountMoreThan"
        count_number = 10
      }
      action {
        type = "expire"
      }
    }
  }
}
```

### Google Artifact Registry (GAR)

```bash
# GOOGLE ARTIFACT REGISTRY

# Authenticate (requires gcloud CLI)
gcloud auth configure-docker <region>-docker.pkg.dev

# Image naming
<region>-docker.pkg.dev/<project>/<repo>/<image>:tag
us-docker.pkg.dev/myproject/myapp/myapp:latest

# Create repository
gcloud artifacts repositories create myapp --repository-format=docker --location=us

# Push image
docker tag myapp:latest us-docker.pkg.dev/myproject/myapp/myapp:latest
docker push us-docker.pkg.dev/myproject/myapp/myapp:latest

# Pull image
docker pull us-docker.pkg.dev/myproject/myapp/myapp:latest

# Vulnerability scanning (automatic)
# GAR provides vulnerability scanning on push
# View scan results in Google Cloud Console

# IAM permissions
# roles/artifactregistry.reader: Pull
# roles/artifactregistry.writer: Push
# roles/artifactregistry.admin: Manage
```

### Azure Container Registry (ACR)

```bash
# AZURE CONTAINER REGISTRY

# Authenticate with Azure CLI
az acr login --name <registry-name>

# Image naming
<registry-name>.azurecr.io/<repo>:tag
myregistry.azurecr.io/myapp:latest

# Create registry
az acr create --resource-group MyRG --name myRegistry --sku Basic

# Push image
docker tag myapp:latest myregistry.azurecr.io/myapp:latest
docker push myregistry.azurecr.io/myapp:latest

# Pull image
docker pull myregistry.azurecr.io/myapp:latest

# Webhooks (trigger CI/CD)
az acr webhook create \
  --registry myRegistry \
  --name build-trigger \
  --actions push \
  --uri https://ci.example.com/webhook

# Geo-replication
az acr replication create \
  --registry myRegistry \
  --destination myRegistryReplica \
  --replications "all"
```

### GitLab Container Registry

```bash
# GITLAB REGISTRY

# Built-in to GitLab
registry.gitlab.com/<group>/<project>/<image>:tag
registry.gitlab.com/mygroup/myproject/myapp:latest

# Authenticate (automatic in GitLab CI)
# Use CI/CD variables automatically

# Push from GitLab CI
build_image:
  image: docker:latest
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD registry.gitlab.com
    - docker build -t registry.gitlab.com/mygroup/myproject/myapp:$CI_COMMIT_SHA .
    - docker push registry.gitlab.com/mygroup/myproject/myapp:$CI_COMMIT_SHA

# Pull image
docker pull registry.gitlab.com/mygroup/myproject/myapp:latest

# Visibility
# Settings → CI/CD → Container Registry
# Private: Requires authentication
# Public: Anyone can pull (rare)

# Cleanup policies
# Settings → Packages & Registries → Cleanup policy
# Auto-delete images older than N days
```

### Harbor (Self-Hosted Registry)

```yaml
# HARBOR: Enterprise-grade self-hosted registry

features:
  - Private and public projects
  - Vulnerability scanning (Trivy integration)
  - Image signing and verification
  - Notary v2 support
  - Replication to other registries
  - RBAC with LDAP/AD/OAuth
  - Artifact retention policies
  - High availability (with multiple replicas)

# Installation (Helm chart)
helm repo add harbor https://helm.goharbor.io/chart
helm install harbor harbor/harbor \
  --set expose.tls.enabled=true \
  --set persistence.enabled=true \
  --set externalURL=harbor.company.com

# Create project via UI or API
# Push to Harbor
docker login harbor.company.com
docker tag myapp:latest harbor.company.com/myproject/myapp:latest
docker push harbor.company.com/myproject/myapp:latest

# Pull from Harbor
docker pull harbor.company.com/myproject/myapp:latest

# Replication to other registries
# Harbor → Docker Hub / ECR / GCR
# Scheduled or trigger-based
```

### Registry Comparison

```yaml
comparison:
  |                 | Docker Hub | GHCR     | ECR      | GAR       | ACR       | GitLab   |
  |-----------------|-----------|-----------|----------|----------|----------|----------|----------|
  | Public          | Yes       | Yes       | No       | No       | No       | Yes      |
  | Private         | Yes       | Yes       | Yes      | Yes      | Yes      | Yes      |
  | Free Tier       | Yes       | Yes       | Yes      | Yes      | Yes      | Yes      |
  | Rate Limits     | High      | High      | None     | None     | None     | Medium   |
  | CD Integration  | Good      | Excellent | Great    | Great    | Great    | Great    |
  | Vuln Scanning   | Scout     | Basic     | Native   | Native   | Native   | Basic    |
  | Multi-region     | No        | No        | Yes      | Yes      | Yes      | No       |
  | Self-hosted     | No        | No        | No       | No       | No       | No       |

# RECOMMENDATIONS:
small_teams:
  registry: "GitHub Container Registry"
  reason: "Free, excellent CI integration, no rate limits"
  use_when: "Using GitHub Actions"

enterprise_aws:
  registry: "AWS ECR"
  reason: "Deep AWS integration, IAM, regional availability"
  use_when: "Deploying to AWS services (EKS, ECS, Lambda)"

enterprise_azure:
  registry: "Azure ACR"
  reason: "Azure AD integration, RBAC, geo-replication"
  use_when: "Deploying to Azure services (AKS, Container Instances)"

self_hosted:
  registry: "Harbor"
  reason: "Complete control, scanning, policies, on-prem"
  use_when: "Compliance requires on-prem, air-gapped"
```

### Registry Mirroring

```bash
# DOCKER DAEMON MIRROR CONFIGURATION

# /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://registry-1.docker.io",
    "https://registry-2.docker.io"
  ]
}

# Pull behavior:
# Docker tries mirrors in order
# Falls back to original registry if all fail

# NEXUS / ARTIFACTORY MIRROR
{
  "registry-mirrors": [
    "https://nexus.company.com/repository/docker-group/"
  ]
}

# Benefits:
# - Faster pulls (local cache)
# - Reduced external bandwidth
# - Private registry caching of public images
```

### Registry Security

```yaml
# SECURITY BEST PRACTICES FOR REGISTRIES:

authentication:
  - Use PATs or tokens, not passwords
  - Rotate credentials regularly
  - Use read-only tokens for CI/CD
  - Enable MFA where available

access_control:
  - Default to private repositories
  - Use least-privilege IAM roles
  - Regularly review repository access
  - Implement branch protection rules

scanning:
  - Enable automatic vulnerability scanning
  - Block images with critical CVEs
  - Fix vulnerabilities before promotion
  - Sign images for integrity

compliance:
  - Retain immutable images for audit trails
  - Use lifecycle policies to clean old images
  - Document image provenance
  - Geo-replicate for data residency

secrets:
  - Never store secrets in images
  - Use registry secrets (ECR, K8s secrets)
  - Use BuildKit `--secret` for build-time secrets
```

### Cross-Registry Pushes

```bash
# PUSH TO MULTIPLE REGISTRIES

# After building once, push everywhere
docker build -t myapp:v1.2.3 .

# Tag for each registry
docker tag myapp:v1.2.3 ghcr.io/myorg/myapp:v1.2.3
docker tag myapp:v1.2.3 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.2.3
docker tag myapp:v1.2.3 gcr.io/myproject/myapp:v1.2.3

# Push to all
docker push ghcr.io/myorg/myapp:v1.2.3
docker push 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.2.3
docker push gcr.io/myproject/myapp:v1.2.3

# Use CI/CD for automation
# Parallel pushes, proper auth handling
```

## Interview Questions

### Q1: What are the main alternatives to Docker Hub?
**A:** Popular alternatives include GitHub Container Registry (ghcr.io), AWS ECR, Google Artifact Registry (GAR), Azure Container Registry (ACR), GitLab Container Registry, and self-hosted options like Harbor. Each has unique features for authentication, pricing, and integration.

### Q2: How do you authenticate to GitHub Container Registry?
**A:** Use `echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin` where CR_PAT is a GitHub Personal Access Token with `write:packages` scope. In GitHub Actions, authentication is automatic.

### Q3: What is the naming convention for AWS ECR images?
**A:** ECR images follow: `<account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>:tag`. For example: `123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest`.

### Q4: What is Harbor and when would you use it?
**A:** Harbor is an open-source, self-hosted container registry with enterprise features like vulnerability scanning, RBAC, and image replication. Use it when you need on-premise hosting, air-gapped environments, or complete control over registry policies.

### Q5: How do registry mirrors work?
**A:** Configure `"registry-mirrors"` in `/etc/docker/daemon.json` with a list of mirror URLs. When pulling, Docker tries mirrors first before the original registry, improving speed and reducing bandwidth.

### Q6: How do you push to multiple registries?
**A:** Build the image once, then tag it for each target registry and push each tag. Example: `docker tag myapp:v1 ghcr.io/org/myapp:v1 && docker push ghcr.io/org/myapp:v1`. Use CI/CD pipelines to automate this process.

### Q7: What security considerations apply to container registries?
**A:** Use private repositories by default, implement proper IAM/RBAC access controls, enable vulnerability scanning, never store secrets in images, rotate credentials regularly, and enable MFA where available.

### Q8: How do you automate registry authentication in CI/CD?
**A:** Use platform-specific login actions (like `docker/login-action` for GitHub) that use stored secrets. For AWS ECR, use `aws ecr get-login-password` command. For GitLab, use CI/CD variables automatically injected into jobs.

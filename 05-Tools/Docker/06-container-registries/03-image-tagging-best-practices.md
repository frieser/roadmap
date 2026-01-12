---
tags: ['docker', 'containers', 'registries', 'devops', 'tools', 'roadmap']
---

# Image Tagging Best Practices

## Summary

Image tags are mutable references that point to specific image versions or variants. Unlike digests (immutable SHA256 hashes), tags can be updated to point to different images. Understanding proper tagging conventions is crucial for reproducible deployments, avoiding accidental production updates, and managing image lifecycle effectively.

## Detailed Explanation

### Tag vs Digest

```bash
# TAG (mutable, changes over time)
myapp:latest
myapp:v1.2
myapp:production

# DIGEST (immutable, always same image)
myapp@sha256:abc123def456...
myapp@sha256:789abc456def...

# Tag can change:
# myapp:latest → Points to v1.0.0 today
# myapp:latest → Points to v2.0.0 tomorrow (different image!)

# Digest never changes:
# myapp@sha256:abc123... → Always the exact same image
```

### Semantic Versioning

```yaml
# SEMVER TAGGING PATTERN
pattern: "<MAJOR>.<MINOR>.<PATCH>[-<PRERELEASE>]"

# Examples:
myapp:1.0.0      # First stable release
myapp:1.0.1      # Bug fix
myapp:1.1.0      # New features (backward compatible)
myapp:2.0.0      # Breaking changes
myapp:1.2.0-beta  # Pre-release
myapp:1.2.0-rc.1  # Release candidate

# TAGGING STRATEGY:
strategies:
  single_tag_per_build:
    # BAD: All builds get :latest
    # - No history
    # - Hard to rollback
    # - Accidental production updates

  semantic_tags:
    # GOOD: Tag each meaningful release
    # - Clear version history
    # - Easy rollback
    # - Explicit intent (beta, stable)

  commit_sha_tags:
    # GOOD: Tag with commit SHA for traceability
    # - myapp:abc1234def (short SHA)
    # - Links directly to source code
    # - Reproducible builds

# TAG LIFECYCLE:
lifecycle:
  development:
    tags: ["dev", "feature-xyz", "pr-123"]
    meaning: "Temporary, not for production"
    
  staging:
    tags: ["staging", "v1.2.0-rc.1"]
    meaning: "Testing, pre-production"
    
  production:
    tags: ["latest", "stable", "v1.2.0"]
    meaning: "Released, deployed"
```

### Branch-Based Tagging

```bash
# BRANCH TO TAG MAPPING

# Main branch
git checkout main
docker build -t myapp:latest .
docker build -t myapp:$(git describe --tags --abbrev=0) .
# myapp:latest, myapp:v1.2.3

# Feature branch
git checkout feature/new-auth
docker build -t myapp:feature-new-auth .
# myapp:feature-new-auth (for testing)

# Release branch
git checkout release/v1.2.0
docker build -t myapp:v1.2.0 .
docker build -t myapp:v1.2.0 .
docker push myapp:v1.2.0

# Using commit SHA
COMMIT_SHA=$(git rev-parse --short HEAD)
docker build -t myapp:${COMMIT_SHA} .
# myapp:abc1234d (traceable to source)
```

```yaml
# GITLAB CI EXAMPLE
build_image:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  # Tags with commit SHA for traceability
  # Tags: myapp:abc1234, myapp:latest
```

### Multi-Architecture Tags

```bash
# TAGGING FOR MULTI-PLATFORM IMAGES

# Build for multiple architectures
docker buildx build --platform linux/amd64 -t myapp:amd64-v1.0 .
docker buildx build --platform linux/arm64 -t myapp:arm64-v1.0 .

# Tag architecture in image name
docker push myapp:amd64-v1.0
docker push myapp:arm64-v1.0

# Create manifest list
docker buildx imagetools create-manifest myapp:v1.0 \
  myapp:amd64-v1.0 \
  myapp:arm64-v1.0

# Push manifest (multi-arch image)
docker push myapp:v1.0
# Single tag works for all architectures!

# Common pattern:
myapp:latest          # Manifest (auto-detects architecture)
myapp:amd64           # Explicit AMD64
myapp:arm64           # Explicit ARM64 (Apple Silicon, ARM servers)
myapp:v1.0            # Manifest
myapp:v1.0-amd64     # Explicit AMD64 version
```

### Environment-Specific Tags

```bash
# ENVIRONMENT TAGS

# Development builds
docker build -t myapp:dev .
docker build -t myapp:dev-${USER} .

# Staging
docker build -t myapp:staging .
docker build -t myapp:staging-${CI_PIPELINE_ID} .

# Production
docker build -t myapp:production .
docker build -t myapp:prod .

# Rollback tags
docker tag myapp:production myapp:prod-2024-01-01
# myapp:prod-2024-01-01 (previous production)

# Promote staging to production
docker tag myapp:staging-v1.2.0-rc.1 myapp:production
docker push myapp:production
```

```yaml
# DOCKER COMPOSE WITH PROFILES
version: '3.8'

services:
  app:
    image: myapp:dev
    # Default to dev when no profile specified

  app-prod:
    image: myapp:production
    profiles:
      - production
    # Only started with --profile production

  app-staging:
    image: myapp:staging
    profiles:
      - staging
```

### The "latest" Tag Trap

```yaml
# WHY "LATEST" IS PROBLEMATIC IN PRODUCTION

problems:
  unexpected_updates:
    issue: "Latest tag moved while you weren't watching"
    consequence: "New (possibly broken) version deployed to production"
    
  no_rollback:
    issue: "Don't know which version was running"
    consequence: "Can't rollback to known-good version"
    
  non_reproducible:
    issue: "Team pulls latest, gets different versions"
    consequence: "Inconsistent environments, hard to debug"
    
  cache_invalidated:
    issue: "Every pull may be different image"
    consequence: "No caching benefit, slower deployments"

# BETTER ALTERNATIVES:
alternatives:
  stable_tag:
    tag: "myapp:stable"
    description: "Only updated for stable releases"
    
  semantic_tags:
    tag: "myapp:v1.2.3"
    description: "Clear version number, never re-assigned"
    
  digest_tags:
    tag: "myapp@sha256:abc123..."
    description: "Immutable reference to exact image"
```

```bash
# PRODUCTION DEPLOYMENT PATTERN

# BAD - Using latest
docker run -d myapp:latest
# Could be new version, break production

# GOOD - Using specific version
docker run -d myapp:v1.2.3
# Known version, tested, documented

# BETTER - Using digest
docker run -d myapp@sha256:abc123def456...
# Exact image, no ambiguity
```

### Retiring Old Tags

```bash
# MANAGING OLD TAGS

# View all tags for an image
docker image ls --format "{{.Repository}}: {{.Tag}}" myapp
# myapp:latest
# myapp:v1.0.0
# myapp:v1.0.1
# myapp:v1.1.0
# ...

# Delete specific old tag (local)
docker rmi myapp:v1.0.0

# Delete specific tag (remote)
# Requires API calls or web UI
# Not supported via CLI directly

# Use garbage collection
# Registry may auto-delete old tags
# Configure retention policies
```

```yaml
# GITLAB CI: CLEANUP OLD IMAGES
cleanup_old_images:
  stage: cleanup
  script:
    - |
      # Keep last 10 tags
      TAGS_TO_DELETE=$(git tag --list | tail -n +11)
      for tag in $TAGS_TO_DELETE; do
        docker rmi myapp:$tag || true
      done
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

### Registry-Specific Tagging

```bash
# GITHUB CONTAINER REGISTRY

# Tag naming
ghcr.io/owner/repo:tag
ghcr.io/owner/repo:latest
ghcr.io/owner/repo:sha-<commit>

# View tags (GitHub Packages UI)
# https://github.com/owner/repo/pkages/container/myapp
```

```bash
# AWS ECR TAGGING

# ECR automatically tags all pushed images
docker tag myapp:v1.2.3 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.2.3
docker tag myapp:v1.2.3 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Lifecycle policy for old tags
aws ecr put-lifecycle-policy \
  --repository-name myapp \
  --policy-text '{
    "rules": [{
      "rulePriority": 1,
      "description": "Keep last 10 images",
      "selection": {
        "tagStatus": "tagged",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {"type": "expire"}
    }]
  }'
```

### Tag Promotion Workflow

```bash
# PROMOTION THROUGH ENVIRONMENTS

# 1. Build in CI
docker build -t myapp:${CI_COMMIT_SHA} .

# 2. Tag as development
docker tag myapp:${CI_COMMIT_SHA} myapp:dev
docker push myapp:dev

# 3. Test development version
# (automated tests pass)

# 4. Tag as staging
docker tag myapp:${CI_COMMIT_SHA} myapp:staging-${CI_COMMIT_SHA}
docker push myapp:staging-${CI_COMMIT_SHA}

# 5. Deploy to staging
# (staging deployment succeeds)

# 6. Tag as production
docker tag myapp:${CI_COMMIT_SHA} myapp:production
docker push myapp:production

# 7. Deploy to production
# (production deployment)

# Result: Same image ID, three tags
# Development, Staging, Production environments
```

### Security Considerations

```yaml
# TAGGING SECURITY

# 1. NEVER TAG IN SECRETS
bad:
  - "myapp:secret-api-key"
  - "myapp:prod-credentials"
  - "myapp:password-abc123"

# 2. SCAN BEFORE TAGGING
workflow:
  - "Build image"
  - "Scan for vulnerabilities"
  - "If clean → Tag as production"
  - "If CVEs found → Fix, rebuild, re-tag"

# 3. USE DIGESTS FOR CRITICAL DEPLOYMENTS
critical_systems:
  - "Financial systems"
  - "Healthcare systems"
  - "Security-sensitive services"
  
  use:
    - "docker run myapp@sha256:abc123..."
    - "Never use latest in production"

# 4. TAG PROMOTION APPROVAL
governance:
  - "Only authorized users can push to production tag"
  - "Require manual approval in CI/CD"
  - "Document tag changes in change log"
```

### Best Practices Summary

```yaml
# COMPREHENSIVE TAGGING BEST PRACTICES

always_do:
  - "Use semantic versioning (v1.2.3)"
  - "Tag with commit SHA for traceability"
  - "Use architecture-specific tags when needed (:amd64, :arm64)"
  - "Use environment tags (dev, staging, prod)"
  - "Use digests for immutable deployments"
  
never_do:
  - "Use :latest in production"
  - "Reuse production tags for development"
  - "Tag with build numbers only (no semantic meaning)"
  - "Include secrets or sensitive data in tags"
  
for_ci_cd:
  - "Automate tagging based on branch/commit"
  - "Promote tags through environments (dev → staging → prod)"
  - "Implement vulnerability scanning before production tagging"
  - "Use registry retention policies to clean old tags"
  
for_release:
  - "Create Git tag matching Docker tag"
  - "Document changes in release notes"
  - "Update :latest and :stable tags only for final releases"
  - "Create rollback tags (prod-YYYY-MM-DD)"
```

## Interview Questions

### Q1: What is the difference between a tag and a digest?
**A:** A tag is a mutable human-readable reference (like `v1.2.3` or `latest`) that can be updated to point to different images. A digest is an immutable SHA256 hash that always points to the exact same image content.

### Q2: Why is using `:latest` discouraged in production?
**A:** The `:latest` tag can be updated at any time, potentially breaking production with unintended changes. It also makes rollbacks difficult and caching unreliable. Use specific version tags or digests for production.

### Q3: What is semantic versioning for container images?
**A:** Semantic versioning follows `<MAJOR>.<MINOR>.<PATCH>` format (e.g., `v1.2.3`). Major = breaking changes, Minor = new features, Patch = bug fixes. This provides clear version history and communication.

### Q4: How do you tag images for multiple architectures?
**A:** Use architecture-specific suffixes like `myapp:amd64-v1.0` and `myapp:arm64-v1.0`, then create a manifest with `docker buildx imagetools create-manifest myapp:v1.0 ...`. The manifest acts as a multi-arch tag.

### Q5: What is a promotion workflow for Docker tags?
**A:** A promotion workflow is building an image once with a unique identifier (commit SHA), then tagging it for different environments (dev, staging, prod) as it passes tests. The same image ID gets multiple tags representing its journey.

### Q6: How do you handle rollback with Docker tags?
**A:** Keep previous production tags (like `prod-2024-01-01`) or use version tags that are never re-assigned (semantic versioning). When you need to rollback, simply deploy the known-good tag instead of the current one.

### Q7: What are lifecycle policies in container registries?
**A:** Lifecycle policies are registry rules that automatically delete old images based on criteria like tag count, age, or pattern. For example, "Keep last 10 images tagged with v" or "Delete images older than 90 days."

### Q8: How should you integrate tagging with CI/CD pipelines?
**A:** Automate tagging based on branch/commit SHA. Use environment-specific tags for deployments. Implement promotion through stages (dev → staging → prod). Scan images for vulnerabilities before tagging as production. Never use `:latest` in production deployments.

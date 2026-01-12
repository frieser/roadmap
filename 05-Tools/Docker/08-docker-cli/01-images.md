---
tags: ['docker', 'containers', 'cli', 'images', 'devops', 'tools', 'roadmap']
---

# Docker Images CLI

## Summary

Docker images are read-only templates that contain all files needed to run an application. The Docker CLI provides comprehensive commands for listing, building, tagging, removing, and managing images. Understanding image lifecycle, layer management, and best practices is crucial for efficient container workflows.

## Detailed Explanation

### Listing Images

```bash
# LIST ALL IMAGES
docker images
# REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
# nginx        latest    abc123def456  2 hours ago     187MB
# postgres      15        ghi789abc012  1 day ago      371MB

# FILTERED LISTING
docker images nginx
docker images "*python*"
docker images --filter "before=24h"

# FORMATTING OPTIONS
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.CreatedAt}}\t{{.Size}}"
docker images --format "{{.Repository}}: {{.Tag}} - {{.Size}}"
docker images --format json | jq

# ONLY SHOW IMAGE IDs
docker images -q
docker images -q nginx

# QUIET MODE (minimal output)
docker images -q --no-trunc
```

```bash
# ADVANCED FILTERING

# By time created
docker images --filter "since=24h"  # Images created in last 24 hours
docker images --filter "until=7d"   # Images created more than 7 days ago

# By label
docker images --filter "label=maintainer=team@example.com"
docker images --filter "label=env=production"

# By reference
docker images --filter "reference=myapp:v1.2.3"
docker images --filter "reference=myregistry.com/*"

# dangling images (no tag)
docker images --filter "dangling=true"

# Combination
docker images --filter "dangling=false" --filter "since=168h"

# ALL FILTERS
docker images --filter "dangling=false" --filter "before=2024-01-01" --filter "label=env=production"
```

### Pulling Images

```bash
# PULL FROM REGISTRY (default: Docker Hub)
docker pull nginx
# Using default tag (latest)
# Pulling layers and storing locally

# PULL SPECIFIC TAG
docker pull nginx:1.25
docker pull nginx:1.25-alpine

# PULL FROM OTHER REGISTRY
docker pull ghcr.io/owner/myapp:v1.0.0
docker pull registry.gitlab.com/group/project:latest
docker pull 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# PULL BY DIGEST (immutable reference)
docker pull nginx@sha256:abc123def456...
docker pull alpine@sha256:789abc456def

# PULL ALL TAGS
docker pull -a nginx  # Pulls all tags for nginx

# PULL WITH OUTPUT
docker pull --no-cache nginx  # Show download progress
docker pull -q nginx  # Quiet mode

# AUTHENTICATED PULL
echo $PASSWORD | docker login registry.company.com
docker pull registry.company.com/myapp:latest

# Multi-arch pull (BuildKit)
docker buildx pull --platform linux/amd64,linux/arm64 myapp:latest
```

### Building Images

```bash
# BUILD FROM DOCKERFILE IN CURRENT DIRECTORY
docker build -t myapp:v1 .
docker build -t myapp:v1.0.0 .

# BUILD FROM DIFFERENT CONTEXT
docker build -t myapp -f /path/to/Dockerfile .
docker build -t myapp -f docker-compose.prod.yml .

# BUILD WITH BUILD ARGUMENTS
docker build --build-arg VERSION=1.0.0 -t myapp:v1.0.0 .
# ARGs defined in Dockerfile: ARG VERSION
# Use --build-arg for each

# BUILD WITH CACHE CONTROL
docker build --no-cache -t myapp:v1 .       # Disable cache
docker build --cache-from=myregistry/cache -t myapp:v1 .
docker build --pull -t myapp:v1 .             # Pull base image

# BUILD WITH OUTPUT
docker build --progress=plain -t myapp:v1 .  # Detailed output
docker build --quiet -t myapp:v1 .        # Minimal output

# MULTI-PLATFORM BUILD (BuildKit)
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
# Builds for multiple architectures

# BUILD WITH SECRETS
docker build --secret id=mysecret -t myapp:v1 .
# In Dockerfile: --mount=type=secret,id=mysecret,target=/app/secret

# BUILD WITH BUILDKIT FEATURES
DOCKER_BUILDKIT=1 docker build -t myapp:v1 .
# Enables parallel builds, mount caching
```

### Tagging Images

```bash
# TAG IMAGE (CREATE NEW TAG)
docker tag nginx:latest myregistry/nginx:stable
docker tag nginx:1.25 myregistry/nginx:v1.25

# TAG WITH MULTIPLE TAGS
docker tag myapp:v1.0 myapp:latest
docker tag myapp:v1.0 myapp:stable

# TAG FOR DIFFERENT REGISTRY
docker tag myapp:v1.0 ghcr.io/owner/myapp:v1.0.0
docker tag myapp:v1.0 registry.company.com/myapp:production

# OVERWRITE EXISTING TAG
docker tag myapp:new-version myapp:latest  # Moves latest pointer

# TAG ALL TAGS TO REGISTRY
docker tag myapp:v1.0 myregistry.com/myapp:prod
docker tag myapp:v1.0 myregistry.com/myapp:latest
docker tag myapp:v1.0 myregistry.com/myapp:v1

# LIST ALL TAGS FOR AN IMAGE
docker images --format "{{.Repository}}: {{.Tag}}" myapp
```

### Pushing Images

```bash
# PUSH TO REGISTRY (default: Docker Hub)
docker push myregistry/myapp:latest

# PUSH SPECIFIC TAG
docker push myregistry/myapp:v1.2.3

# PUSH ALL TAGS
docker push -a myregistry/myapp
# Pushes all tags: latest, v1.0, v1.1, etc.

# PUSH TO OTHER REGISTRY
docker push ghcr.io/owner/myapp:v1.0.0
docker push 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# AUTHENTICATED PUSH
echo $TOKEN | docker login registry.company.com -u username --password-stdin
docker push registry.company.com/myapp:latest

# PUSH WITH OUTPUT
docker push --quiet myregistry/myapp:latest
docker push --disable-content-trust myregistry/myapp:latest
```

### Removing Images

```bash
# REMOVE BY IMAGE NAME
docker rmi nginx
docker rmi myapp:v1.2.3

# REMOVE BY IMAGE ID
docker rmi abc123def456
docker rmi $(docker images -q myapp)  # Get ID, then remove

# FORCE REMOVE (RUNNING CONTAINERS)
docker rmi -f nginx
docker rmi -f $(docker images -q myapp)

# REMOVE MULTIPLE IMAGES
docker rmi nginx:1.25 nginx:1.24 postgres:15
docker rmi $(docker images -q "*old*")

# REMOVE ALL DANGLING IMAGES
docker image prune  # Remove all images not tagged and not used
docker image prune -a  # Remove all unused images (dangling or not referenced)

# REMOVE UNUSED IMAGES
docker image prune -f --filter "until=24h"
docker image prune -f --filter "with Label=env=staging"

# REMOVE BY FILTER
docker image prune -a --filter "dangling=true"
docker image prune -a --filter "until=7d"

# PRUNE WITH PROMPT
docker image prune  # Ask for confirmation
docker image prune -f  # Force without confirmation

# DRY RUN (SHOW WHAT WOULD BE DELETED)
docker image prune --dry-run
```

### Inspecting Images

```bash
# DETAILED INSPECTION
docker inspect myapp:latest
# Shows full JSON configuration
docker inspect myapp:latest | jq '.[0]'

# INSPECT SPECIFIC FIELD
docker inspect --format='{{.Created}}' myapp:latest
docker inspect --format='{{.Architecture}}' myapp:latest

# INSPECT MULTIPLE FIELDS
docker inspect --format='{{.RepoTags}} - {{.Size}} - {{.Id}}' myapp:latest

# SHOW HISTORY (LAYERS)
docker history myapp:latest
# IMAGE        CREATED BY        CREATED          SIZE      COMMENT
# abc123       Dockerfile   2 hours ago     187MB     CMD ["nginx", "-g..."]

# INSPECT LAYERS IN DETAIL
docker history --no-trunc myapp:latest
docker history --format "table {{.CreatedBy}}\t{{.Size}}" myapp:latest

# VIEW IMAGE MANIFEST
docker manifest inspect myapp:latest
# Shows architecture, OS, layers for all platforms
```

### Import/Export Images

```bash
# EXPORT IMAGE TO TAR FILE
docker save -o myapp.tar myapp:latest
docker save -o images.tar myapp:latest postgres:15 nginx
docker save -o backup.tar $(docker images -q "*myapp*")

# EXPORT MULTIPLE IMAGES
docker save -o all-images.tar myapp:latest postgres:15 nginx

# LOAD IMAGE FROM TAR FILE
docker load -i myapp.tar
docker load < all-images.tar

# IMPORT FROM TAR STREAM
docker load < myapp.tar.gz

# IMPORT FROM CONTAINER EXPORT
docker import myapp-container-export.tar new-image:latest

# SHOW WHAT WAS LOADED
docker images  # New images appear
```

### Image Layers and Size

```bash
# VIEW LAYER INFORMATION
docker history myapp:latest
# Layer 1 (base)
# Layer 2 (dependencies)
# Layer 3 (application)

# VIEW LAYER SIZES
docker history --format "table {{.Size}}" myapp:latest

# CALCULATE TOTAL SIZE
docker images --format "{{.Size}}" myapp:latest | awk '{sum+=$1} END {print sum}'

# FIND LARGE IMAGES
docker images --format "{{.Size}} {{.Repository}}:{{.Tag}}" | sort -rh

# FIND DANGLING IMAGES (wasting space)
docker images --filter "dangling=true" --format "{{.Size}}" | awk '{sum+=$1} END {print sum "bytes"}'

# REMOVE LAYERS TO REDUCE SIZE
docker image prune -a  # Remove all unused images
docker builder prune  # Remove BuildKit cache
```

### Search Images

```bash
# SEARCH DOCKER HUB (DEFAULT REGISTRY)
docker search nginx
# NAME                  DESCRIPTION                                     STARS     OFFICIAL
# nginx                 Official build of Nginx.                      18000     [OK]

# SEARCH WITH LIMIT
docker search --limit 10 nginx

# SEARCH AUTOMATED BUILDS
docker search --filter is-automated=true

# SEARCH ONLY OFFICIAL
docker search --filter is-official=true nginx

# SEARCH BY STARS
docker search --filter stars=50+ nginx

# FORMATTING SEARCH RESULTS
docker search --format "table {{.Name}}\t{{.StarCount}}\t{{.Description}}" nginx
```

### Content Trust

```bash
# ENABLE DOCKER CONTENT TRUST
export DOCKER_CONTENT_TRUST=1

# SIGN IMAGE (when pushing)
docker trust sign myregistry/myapp:v1.0.0

# PUSH TRUSTED IMAGE
docker push --sign myregistry/myapp:v1.0.0
# Prompts for key passphrase
# Image signed and verifiable

# VERIFY TRUSTED IMAGE
docker trust inspect myregistry/myapp:v1.0.0
docker trust verify myregistry/myapp:v1.0.0

# ADD SIGNER
docker trust signer add --key mykey.pem myregistry/myapp:v1.0.0

# DISABLE CONTENT TRUST
unset DOCKER_CONTENT_TRUST
export DOCKER_CONTENT_TRUST=0
```

### Best Practices

```bash
# IMAGE MANAGEMENT BEST PRACTICES

# 1. USE SPECIFIC TAGS IN PRODUCTION
# BAD: docker run myapp:latest
# GOOD: docker run myapp:v1.2.3

# 2. CLEAN UP DANGLING IMAGES REGULARLY
docker image prune -f
# Schedule in CI/CD pipelines

# 3. USE MULTI-STAGE BUILDS TO REDUCE SIZE
# Build stages keep only runtime dependencies in final image

# 4. TAG WITH COMMIT SHA FOR TRACEABILITY
COMMIT_SHA=$(git rev-parse --short HEAD)
docker tag myapp:${COMMIT_SHA} myapp:dev

# 5. SCAN IMAGES FOR VULNERABILITIES
docker scout cves myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# 6. USE CONTENT TRUST FOR CRITICAL DEPLOYMENTS
docker trust sign and push production images

# 7. PULL WITH DIGEST FOR IMMUTABILITY
docker pull myapp@sha256:abc123...
# Never changes, always exact same image
```

## Interview Questions

### Q1: What is the difference between `docker images` and `docker image ls`?
**A:** `docker images` and `docker image ls` are aliases - they do the same thing. Both list Docker images stored locally with their tags, sizes, and creation times. Use whichever you prefer.

### Q2: What are dangling images and how do you remove them?
**A:** Dangling images are images that are not tagged and are not referenced by any other image (usually failed builds). They waste disk space. Remove them with `docker image prune` or `docker rmi $(docker images -f "dangling=true" -q)`.

### Q3: How do you tag an image for a different registry?
**A:** Use `docker tag source_image target_image`. For example: `docker tag myapp:v1.0 ghcr.io/owner/myapp:v1.0`. This creates a new reference pointing to the same image ID.

### Q4: What is the difference between `docker save` and `docker export`?
**A:** `docker save` creates a tar of an image (including all layers and metadata), suitable for backup or transfer. `docker export` creates a tar of a container's filesystem (single layer), losing history and metadata.

### Q5: How do you pull an image by digest instead of tag?
**A:** Use `docker pull image@sha256:abc123...` where you replace `abc123...` with the full digest. This always pulls the exact same image since digests are immutable, unlike tags which can change.

### Q6: What does `--no-cache` do when building images?
**A:** The `--no-cache` flag disables Docker's layer caching, forcing all layers to be rebuilt from scratch. This ensures a fresh build but is slower and wastes cache benefits. Use only when you need a guaranteed clean build.

### Q7: How do you remove all unused images efficiently?
**A:** Use `docker image prune -a` to remove all images not referenced by any container. Use `docker image prune --filter "until=7d"` to remove images older than 7 days. Always use `-f` to skip confirmation prompts.

### Q8: What is Docker Content Trust and when would you use it?
**A:** Docker Content Trust provides digital signing and verification of images to ensure they come from trusted sources and haven't been tampered with. Enable with `export DOCKER_CONTENT_TRUST=1` and sign images with `docker trust sign`. Use for critical deployments requiring supply chain security.

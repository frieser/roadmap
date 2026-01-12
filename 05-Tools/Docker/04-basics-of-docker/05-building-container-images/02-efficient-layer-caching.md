---
tags: ['docker', 'containers', 'optimization', 'devops', 'tools', 'roadmap']
---

# Efficient Layer Caching

## Summary

Docker builds images in layers, caching each layer. When a layer changes, all subsequent layers are rebuilt. Understanding this caching mechanism is crucial for fast builds. Key strategies include ordering instructions from least to most frequently changing, separating dependency installation from code copying, using `.dockerignore`, and leveraging BuildKit cache mounts.

## Detailed Explanation

### How Layer Caching Works

```
Dockerfile:
FROM node:20              ┐
                          │ Cached if unchanged
RUN apt-get update        │
                          ├─ Cached (base layers)
COPY package.json ./      ┘
                          ┐
RUN npm install           │ Invalidated when package.json changes
                          │ Then EVERYTHING below rebuilds
COPY . .                  │
                          ├─ Rebuild zone
RUN npm run build         │
                          │
CMD ["npm", "start"]      ┘
```

```bash
# First build - all layers created
docker build -t myapp:v1 .
# Step 2/6: FROM node:20              [Building]
# Step 3/6: RUN apt-get update        [Building]
# Step 4/6: COPY package.json         [Building]
# Step 5/6: RUN npm install           [Building]
# Step 6/6: COPY . .                  [Building]

# Second build - source code changed only
docker build -t myapp:v2 .
# Step 2/6: FROM node:20              [Cached]
# Step 3/6: RUN apt-get update        [Cached]
# Step 4/6: COPY package.json         [Cached]    <- Still same
# Step 5/6: RUN npm install           [Cached]    <- Still valid
# Step 6/6: COPY . .                  [Building]  <- Changed!

# Third build - package.json changed
docker build -t myapp:v3 .
# Step 2/6: FROM node:20              [Cached]
# Step 3/6: RUN apt-get update        [Cached]
# Step 4/6: COPY package.json         [Building]  <- Changed!
# Step 5/6: RUN npm install           [Building]  <- Must rebuild
# Step 6/6: COPY . .                  [Building]  <- Must rebuild
```

### Cache Invalidation Rules

```yaml
# What invalidates cache:

layer_type: FROM
  invalidated_by:
    - Different image/tag
    - Image updated in registry (with --pull)

layer_type: RUN
  invalidated_by:
    - Command string changed
    - Previous layer invalidated
  note: "Content of external URLs not checked"

layer_type: COPY/ADD
  invalidated_by:
    - File content changed (checksum)
    - File permissions changed
    - Previous layer invalidated
  note: "Modification time ignored"

layer_type: ARG
  invalidated_by:
    - ARG value changed (if used in subsequent RUN)
```

### Ordering Instructions Correctly

```dockerfile
# BAD - Code changes invalidate everything
FROM python:3.11
COPY . /app/                        # Changes frequently
WORKDIR /app
RUN pip install -r requirements.txt # Reinstalls every time!
CMD ["python", "app.py"]

# GOOD - Dependencies cached separately
FROM python:3.11
WORKDIR /app
COPY requirements.txt .             # Changes rarely
RUN pip install -r requirements.txt # Cached usually
COPY . /app/                        # Changes frequently
CMD ["python", "app.py"]
```

```dockerfile
# Node.js optimal ordering
FROM node:20-alpine

WORKDIR /app

# 1. Package files (change rarely)
COPY package.json package-lock.json ./

# 2. Install dependencies (cached when package*.json unchanged)
RUN npm ci --only=production

# 3. Source code (changes frequently)
COPY . .

# 4. Build (only runs when source changes)
RUN npm run build

CMD ["node", "dist/index.js"]
```

### Multi-Stage Cache Optimization

```dockerfile
# syntax=docker/dockerfile:1

# Stage: dependencies (cached separately)
FROM node:20 AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Stage: builder (uses cached deps)
FROM node:20 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage: production
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]

# Benefits:
# - deps stage cached even when source changes
# - Can rebuild builder without reinstalling dependencies
```

### BuildKit Cache Mounts

```dockerfile
# syntax=docker/dockerfile:1

# Cache apt packages
FROM ubuntu:22.04
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && apt-get install -y python3 python3-pip

# Cache pip packages
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Cache npm packages
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Cache Go modules
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -o app .

# Cache Maven
RUN --mount=type=cache,target=/root/.m2/repository \
    mvn package

# Cache Gradle
RUN --mount=type=cache,target=/root/.gradle \
    gradle build
```

### Bind Mounts for Build Context

```dockerfile
# syntax=docker/dockerfile:1

# Mount source code instead of copying (for builds)
FROM golang:1.21 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
# Mount source, only copy what's needed
RUN --mount=type=bind,source=.,target=/build,rw \
    go build -o /app

FROM alpine:3.18
COPY --from=builder /app /app
CMD ["/app"]
```

### .dockerignore for Faster Builds

```bash
# .dockerignore - Exclude from build context

# Without .dockerignore:
# - Entire directory sent to Docker daemon
# - Larger context = slower builds
# - Unnecessary cache invalidation

# Version control
.git
.gitignore
.svn

# Dependencies (installed in container)
node_modules
vendor
__pycache__
.venv

# Build artifacts
dist
build
target
*.o
*.pyc

# IDE
.idea
.vscode
*.swp
*.swo

# Tests (usually not needed in production image)
**/test
**/tests
**/__tests__
*.test.js
*.spec.js

# Documentation
*.md
docs/

# Local configuration
.env*
!.env.example
*.local

# Docker files
Dockerfile*
docker-compose*
.docker

# Cache invalidation prevention
.git        # Prevents cache busting from .git changes
npm-debug.log
yarn-debug.log
yarn-error.log
```

### Parallelizing Builds

```dockerfile
# syntax=docker/dockerfile:1

# Parallel stages (BuildKit builds independent stages concurrently)
FROM node:20 AS frontend
WORKDIR /frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

FROM golang:1.21 AS backend
WORKDIR /backend
COPY backend/go.* ./
RUN go mod download
COPY backend/ .
RUN go build -o api

FROM python:3.11 AS ml-service
WORKDIR /ml
COPY ml/requirements.txt .
RUN pip install -r requirements.txt
COPY ml/ .

# Final stage - combines all
FROM alpine:3.18
COPY --from=frontend /frontend/dist /app/static
COPY --from=backend /backend/api /app/api
COPY --from=ml-service /ml /app/ml
CMD ["/app/api"]

# BuildKit builds frontend, backend, ml-service in PARALLEL
# Final stage waits for all to complete
```

### Cache Management Commands

```bash
# View build cache
docker buildx du

# Prune build cache
docker builder prune

# Prune all cache
docker builder prune -a

# Keep only recent cache
docker builder prune --keep-storage 10GB

# Force fresh build (no cache)
docker build --no-cache -t myapp .

# Pull base image before build
docker build --pull -t myapp .

# Export/import cache (CI/CD)
docker buildx build \
  --cache-from type=registry,ref=myregistry/myapp:cache \
  --cache-to type=registry,ref=myregistry/myapp:cache,mode=max \
  -t myapp:latest .
```

### CI/CD Cache Strategies

```yaml
# GitHub Actions with cache
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build with cache
  uses: docker/build-push-action@v5
  with:
    context: .
    cache-from: type=gha
    cache-to: type=gha,mode=max
    push: true
    tags: myapp:latest

# GitLab CI with registry cache
build:
  script:
    - docker buildx build
        --cache-from $CI_REGISTRY_IMAGE:cache
        --cache-to $CI_REGISTRY_IMAGE:cache
        --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        --push .
```

### Measuring Build Performance

```bash
# Verbose build output
docker build --progress=plain -t myapp .

# Time the build
time docker build -t myapp .

# Analyze with BuildKit
DOCKER_BUILDKIT=1 docker build -t myapp . 2>&1 | tee build.log

# View layer sizes
docker history myapp

# Detailed image analysis
docker inspect myapp
dive myapp  # Third-party tool for analyzing layers
```

## Interview Questions

### Q1: How does Docker layer caching work?
**A:** Docker caches each layer and reuses it if unchanged. When a layer changes, all subsequent layers must be rebuilt. Docker checks for changes by comparing instruction text (RUN) or file checksums (COPY/ADD).

### Q2: Why should you COPY package.json before the rest of your code?
**A:** Because dependencies change less frequently than source code. By copying package.json first and running npm install, that layer stays cached even when source code changes. Only the final COPY layer rebuilds.

### Q3: What is a BuildKit cache mount?
**A:** Cache mounts (`--mount=type=cache`) persist data between builds without including it in the image. Useful for package manager caches (apt, npm, pip) - packages don't re-download on every build but aren't in the final image.

### Q4: How does .dockerignore improve build performance?
**A:** It excludes files from the build context sent to Docker daemon. Smaller context means faster transfer and fewer files to check for COPY cache invalidation. It also prevents accidental inclusion of large directories like node_modules or .git.

### Q5: How can BuildKit parallelize builds?
**A:** BuildKit analyzes the Dockerfile dependency graph and builds independent stages concurrently. Stages that don't depend on each other run in parallel, while dependent stages wait for their dependencies.

### Q6: What causes cache invalidation for a RUN instruction?
**A:** The cache is invalidated if the command string changes or if any previous layer was invalidated. The actual content fetched by the command (like apt packages) is NOT checked - only the instruction text.

### Q7: How do you share build cache in CI/CD pipelines?
**A:** Use registry cache: `--cache-from` and `--cache-to` with `type=registry`. For GitHub Actions, use `type=gha`. This stores cache externally so subsequent pipeline runs can reuse layers.

### Q8: What is the difference between `--no-cache` and `--pull`?
**A:** `--no-cache` ignores the local build cache, rebuilding every layer from scratch. `--pull` fetches the latest version of the base image (FROM) but still uses cache for other layers if valid.

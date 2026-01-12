---
tags: ['docker', 'containers', 'dockerfile', 'devops', 'tools', 'roadmap']
---

# Dockerfiles

## Summary

A Dockerfile is a text file containing instructions to build a Docker image. Each instruction creates a layer in the image, defining the base image, installed software, configuration, and startup command. Understanding Dockerfile syntax and best practices is essential for creating efficient, secure, and maintainable container images.

## Detailed Explanation

### Basic Dockerfile Structure

```dockerfile
# Comment - Dockerfiles support comments
# Syntax (optional but recommended for BuildKit features)
# syntax=docker/dockerfile:1

# Base image - REQUIRED, always first instruction
FROM ubuntu:22.04

# Metadata labels
LABEL maintainer="team@example.com"
LABEL version="1.0"
LABEL description="My application container"

# Set environment variables
ENV APP_HOME=/app
ENV NODE_ENV=production

# Set working directory
WORKDIR $APP_HOME

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Copy files from build context
COPY package*.json ./
COPY src/ ./src/

# Run commands during build
RUN npm install --production

# Expose port documentation
EXPOSE 3000

# Define volume mount point
VOLUME /app/data

# Create non-root user
RUN useradd -r -u 1001 appuser
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Default command
CMD ["node", "src/index.js"]
```

### Key Instructions

```dockerfile
# FROM - Set base image
FROM ubuntu:22.04
FROM node:20-alpine
FROM scratch  # Empty base for static binaries

# Multi-stage build
FROM node:20 AS builder
WORKDIR /build
COPY . .
RUN npm run build

FROM node:20-alpine
COPY --from=builder /build/dist ./dist
CMD ["node", "dist/index.js"]

# ARG - Build-time variables
ARG VERSION=1.0.0
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}
LABEL version="${VERSION}"

# Build with args
# docker build --build-arg VERSION=2.0.0 .

# ENV - Runtime environment variables
ENV NODE_ENV=production
ENV PORT=3000

# ENV vs ARG:
# ARG: Only available during build
# ENV: Available during build AND runtime
```

### COPY vs ADD

```dockerfile
# COPY - Copy files from build context (preferred)
COPY package.json ./
COPY src/ ./src/
COPY --chown=node:node . .

# ADD - Like COPY but with extra features
# - Auto-extracts tar archives
# - Supports URLs (not recommended)
ADD archive.tar.gz /app/
ADD https://example.com/file.txt /app/  # Avoid - not cached!

# BEST PRACTICE: Use COPY unless you need tar extraction
COPY requirements.txt .
# NOT: ADD requirements.txt .
```

### RUN Instruction

```dockerfile
# Shell form (runs in shell)
RUN apt-get update && apt-get install -y curl

# Exec form (no shell processing)
RUN ["apt-get", "update"]

# Multi-line RUN (combine to reduce layers)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget \
      git && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

# BuildKit cache mounts (for package managers)
# syntax=docker/dockerfile:1
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y curl

RUN --mount=type=cache,target=/root/.npm \
    npm install

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

### CMD vs ENTRYPOINT

```dockerfile
# CMD - Default command (can be overridden)
CMD ["node", "app.js"]
# Override: docker run myimage python script.py

# ENTRYPOINT - Fixed executable
ENTRYPOINT ["node"]
CMD ["app.js"]
# Override CMD: docker run myimage other.js
# Override ENTRYPOINT: docker run --entrypoint python myimage

# Shell form (runs in shell - signals not forwarded properly)
CMD npm start  # Runs as: /bin/sh -c "npm start"

# Exec form (recommended - proper signal handling)
CMD ["npm", "start"]  # Runs directly as PID 1

# Common pattern: ENTRYPOINT + CMD
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["postgres"]
# Runs: docker-entrypoint.sh postgres
# Override: docker run postgres:15 postgres --help
```

### EXPOSE and Networking

```dockerfile
# EXPOSE - Document which ports the container listens on
EXPOSE 80
EXPOSE 443
EXPOSE 3000/tcp
EXPOSE 5000/udp

# EXPOSE is documentation only!
# You still need -p when running:
# docker run -p 8080:80 myimage

# Multiple ports
EXPOSE 80 443 3000
```

### WORKDIR and USER

```dockerfile
# WORKDIR - Set working directory
WORKDIR /app
# Creates directory if it doesn't exist

# Multiple WORKDIR (relative paths work)
WORKDIR /app
WORKDIR src
WORKDIR ../config
# Now in /app/config

# USER - Run commands as this user
RUN useradd -r -u 1001 -g root appuser
USER appuser
# All subsequent commands run as appuser

# Or use existing user
USER nobody
USER 1000:1000  # UID:GID
```

### VOLUME

```dockerfile
# VOLUME - Create mount point for external volumes
VOLUME /app/data
VOLUME ["/var/log", "/var/data"]

# When container runs without explicit mount,
# Docker creates an anonymous volume

# Better practice: Define volumes at runtime
# docker run -v mydata:/app/data myimage
```

### HEALTHCHECK

```dockerfile
# HEALTHCHECK - Container health monitoring
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Options:
# --interval: Time between checks (default 30s)
# --timeout: Check timeout (default 30s)
# --start-period: Initial grace period (default 0s)
# --retries: Failures before unhealthy (default 3)

# Disable parent image's healthcheck
HEALTHCHECK NONE

# Check status
# docker inspect --format='{{.State.Health.Status}}' container
```

### Multi-Stage Builds

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o app .

# Stage 2: Runtime
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /build/app .
USER nobody
EXPOSE 8080
CMD ["./app"]

# Benefits:
# - Build tools not in final image
# - Smaller image size
# - Reduced attack surface
```

```dockerfile
# Multi-stage for Node.js
FROM node:20 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### .dockerignore

```bash
# .dockerignore - exclude files from build context

# Git
.git
.gitignore

# Dependencies (will be installed in container)
node_modules
vendor
__pycache__

# Build outputs
dist
build
*.pyc
*.pyo

# IDE and editors
.idea
.vscode
*.swp

# Environment and secrets
.env
.env.*
*.pem
*.key

# Documentation
*.md
!README.md

# Tests
test
tests
__tests__
*.test.js
*.spec.js

# Docker files (usually not needed in image)
Dockerfile*
docker-compose*
```

### Building Images

```bash
# Basic build
docker build -t myapp:v1 .

# With build args
docker build --build-arg VERSION=1.0.0 -t myapp:v1 .

# Specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# No cache (fresh build)
docker build --no-cache -t myapp:v1 .

# Multi-platform build (BuildKit)
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:v1 .

# Build and push
docker build -t registry.example.com/myapp:v1 --push .

# View build output
docker build --progress=plain -t myapp:v1 .
```

## Interview Questions

### Q1: What is the difference between CMD and ENTRYPOINT?
**A:** ENTRYPOINT sets the fixed executable that always runs. CMD provides default arguments that can be overridden at runtime. Using both, ENTRYPOINT is the command and CMD provides default arguments.

### Q2: Why should you use exec form `["cmd"]` instead of shell form `cmd`?
**A:** Exec form runs the process directly as PID 1, properly receiving signals (SIGTERM for graceful shutdown). Shell form wraps the command in `/bin/sh -c`, meaning the shell is PID 1 and signals may not reach your application.

### Q3: What is the difference between COPY and ADD?
**A:** Both copy files into the image, but ADD has extra features: auto-extracting tar archives and downloading URLs. Use COPY unless you specifically need tar extraction. URL downloads aren't cached and should be avoided.

### Q4: What is a multi-stage build and why use it?
**A:** Multi-stage builds use multiple FROM statements, each creating a new stage. You can copy artifacts from one stage to another. This allows separating build dependencies from runtime, resulting in smaller, more secure final images.

### Q5: What does .dockerignore do?
**A:** It excludes files from the build context sent to Docker daemon. This speeds up builds and prevents accidentally including sensitive files (secrets, .git directory) or unnecessary files (node_modules) in the image.

### Q6: What is the difference between ARG and ENV?
**A:** ARG is only available during build time and can be set with `--build-arg`. ENV is available during both build time and container runtime. Use ARG for build configuration, ENV for runtime configuration.

### Q7: Why combine RUN commands with && instead of using multiple RUN instructions?
**A:** Each RUN creates a new layer. Combining commands reduces layers and, importantly, allows cleanup in the same layer (e.g., `apt-get install && rm -rf /var/lib/apt/lists/*`). Cleaning up in a separate RUN doesn't reduce image size.

### Q8: What happens if you use VOLUME in a Dockerfile?
**A:** It creates a mount point and marks it for anonymous volume attachment. If no volume is explicitly mounted at runtime, Docker creates an anonymous volume. However, it's better practice to define volumes at runtime with `-v`.

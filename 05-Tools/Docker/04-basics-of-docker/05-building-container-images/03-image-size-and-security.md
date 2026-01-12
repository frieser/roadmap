---
tags: ['docker', 'containers', 'security', 'optimization', 'devops', 'tools', 'roadmap']
---

# Image Size and Security

## Summary

Smaller Docker images are faster to build, push, pull, and start. They also have a reduced attack surface with fewer vulnerabilities. Key strategies include using minimal base images (Alpine, distroless), multi-stage builds, removing unnecessary files, and scanning for vulnerabilities. Security best practices include running as non-root, using trusted base images, scanning regularly, and minimizing installed packages.

## Detailed Explanation

### Why Size Matters

```yaml
image_size_impacts:
  build_time:
    - Larger layers take longer to build
    - More to transfer to daemon
  
  push_pull_time:
    - Network transfer (200MB vs 20MB = 10x difference)
    - Registry storage costs
    - CI/CD pipeline duration
  
  startup_time:
    - Kubernetes/Swarm node pulls
    - Auto-scaling speed
    - Cold start latency
  
  security:
    - More packages = more potential vulnerabilities
    - Larger attack surface
    - More to scan and maintain

# Example sizes:
# ubuntu:22.04     ~77MB
# python:3.11      ~1GB
# python:3.11-slim ~150MB
# python:3.11-alpine ~50MB
# distroless/python3 ~50MB
# scratch (empty)   0MB
```

### Choosing Base Images

```dockerfile
# Full OS (avoid for production)
FROM ubuntu:22.04      # ~77MB, many packages, familiar
FROM debian:bookworm   # ~124MB, stable, well-documented

# Slim variants (good balance)
FROM python:3.11-slim  # ~150MB, minimal Debian
FROM node:20-slim      # ~200MB, minimal Debian

# Alpine (smallest, some compatibility issues)
FROM python:3.11-alpine  # ~50MB, musl libc
FROM node:20-alpine      # ~130MB, musl libc

# Distroless (minimal, no shell)
FROM gcr.io/distroless/python3  # ~50MB
FROM gcr.io/distroless/java17   # ~220MB
FROM gcr.io/distroless/nodejs20 # ~130MB

# Scratch (for static binaries only)
FROM scratch
COPY myapp /myapp
CMD ["/myapp"]
```

### Multi-Stage for Minimal Images

```dockerfile
# syntax=docker/dockerfile:1

# Build stage - includes all build tools
FROM golang:1.21 AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o app

# Runtime stage - minimal
FROM scratch
COPY --from=builder /build/app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
USER 1000:1000
ENTRYPOINT ["/app"]

# Result:
# golang:1.21 base: ~800MB
# Final image:      ~5MB (just the binary + certs)
```

```dockerfile
# Python multi-stage
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]

# Saves ~850MB by not including build tools in final image
```

### Reducing Layer Size

```dockerfile
# BAD - Large layers, cache not cleaned
RUN apt-get update
RUN apt-get install -y curl wget git
RUN rm -rf /var/lib/apt/lists/*  # Doesn't reduce size!

# GOOD - Clean in same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

# Key techniques:
# --no-install-recommends  Skip suggested packages
# rm -rf /var/lib/apt/lists/*  Remove apt cache
# --no-cache (apk)  Don't cache package index
# --no-cache-dir (pip)  Don't cache pip downloads

# Alpine example
RUN apk add --no-cache \
    python3 \
    py3-pip
```

### Removing Unnecessary Files

```dockerfile
# Remove documentation
RUN rm -rf /usr/share/doc /usr/share/man

# Remove package manager cache
RUN rm -rf /var/lib/apt/lists/* \
    /var/cache/apt/archives/* \
    /root/.cache \
    /tmp/*

# Python: remove .pyc files
RUN find /app -type f -name "*.pyc" -delete && \
    find /app -type d -name "__pycache__" -delete

# Node: production dependencies only
RUN npm ci --only=production && \
    npm cache clean --force

# Multi-stage to exclude dev dependencies
FROM node:20 AS deps
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
COPY --from=deps /node_modules ./node_modules
COPY . .
```

### Security Best Practices

```dockerfile
# 1. Run as non-root user
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser
CMD ["node", "app.js"]

# 2. Use specific versions, not latest
FROM node:20.10.0-alpine3.18  # Pinned version
# NOT: FROM node:latest

# 3. Minimal packages only
RUN apk add --no-cache \
    ca-certificates \
    tzdata
# Don't install: curl, wget, bash (unless needed)

# 4. Read-only filesystem
# At runtime: docker run --read-only myapp

# 5. No secrets in image
# BAD:
COPY .env /app/.env
ENV API_KEY=secret123

# GOOD: Use runtime secrets
# docker run -e API_KEY=$API_KEY myapp

# 6. Use COPY not ADD (unless extracting tar)
COPY app.py /app/
# NOT: ADD app.py /app/

# 7. Verify downloads
RUN curl -fsSL https://example.com/file.tar.gz -o file.tar.gz && \
    echo "abc123... file.tar.gz" | sha256sum -c - && \
    tar xzf file.tar.gz
```

### Distroless Images

```dockerfile
# Distroless: No shell, no package manager
# Only your application and runtime dependencies

# Python distroless
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --target=/app/deps -r requirements.txt
COPY . .

FROM gcr.io/distroless/python3-debian11
WORKDIR /app
COPY --from=builder /app /app
ENV PYTHONPATH=/app/deps
CMD ["app.py"]

# Benefits:
# - No shell for attackers to use
# - Minimal CVEs
# - Small size
# 
# Drawbacks:
# - Can't exec into container for debugging
# - No package manager for quick fixes
```

### Vulnerability Scanning

```bash
# Docker Scout (built-in)
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Trivy (free, popular)
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Grype (Anchore)
grype myapp:latest

# Snyk
snyk container test myapp:latest

# CI/CD integration
# GitHub Actions
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    format: 'sarif'
    exit-code: '1'  # Fail on vulnerabilities
    severity: 'CRITICAL,HIGH'

# Block deployment if vulnerabilities found
```

### Image Signing and Verification

```bash
# Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker push myregistry/myapp:v1  # Signs image
docker pull myregistry/myapp:v1  # Verifies signature

# Cosign (sigstore)
cosign sign myregistry/myapp:v1
cosign verify myregistry/myapp:v1

# Kubernetes policy (Kyverno)
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-signed-images
spec:
  validationFailureAction: enforce
  rules:
  - name: verify-signature
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "myregistry/*"
      attestors:
      - entries:
        - keys:
            publicKeys: |
              -----BEGIN PUBLIC KEY-----
              ...
```

### Analyzing Image Size

```bash
# View layer sizes
docker history myapp:latest

# Detailed breakdown
docker inspect myapp:latest | jq '.[0].Size'

# Dive - interactive layer analysis
dive myapp:latest
# Shows:
# - Layer efficiency
# - Wasted space
# - Files in each layer

# Check for large files
docker run --rm myapp:latest du -sh /* 2>/dev/null | sort -rh

# Compare image sizes
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h
```

### Complete Secure Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# Build stage
FROM python:3.11-slim AS builder

WORKDIR /build

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Production stage
FROM python:3.11-slim

# Security: Create non-root user
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy application
COPY --chown=appuser:appgroup . .

# Security: Switch to non-root user
USER appuser

# Environment
ENV PATH=/home/appuser/.local/bin:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Metadata
LABEL maintainer="team@example.com" \
      version="1.0.0"

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

EXPOSE 8000

CMD ["python", "app.py"]
```

## Interview Questions

### Q1: Why does image size matter for security?
**A:** Larger images contain more packages, which means more potential vulnerabilities. Each additional package increases the attack surface. Minimal images have fewer CVEs and are easier to audit and maintain.

### Q2: What is a distroless image?
**A:** Distroless images contain only the application and its runtime dependencies - no shell, package manager, or other OS utilities. This minimizes attack surface and CVEs but makes debugging harder.

### Q3: How do multi-stage builds reduce image size?
**A:** They separate build-time dependencies from runtime. Build tools, compilers, and dev dependencies stay in build stages. Only necessary artifacts (binaries, installed packages) are copied to the minimal final stage.

### Q4: Why should containers run as non-root?
**A:** Running as root inside a container can be dangerous if there's a container escape vulnerability. Non-root execution limits what an attacker can do even if they compromise the container process.

### Q5: Why clean up in the same RUN instruction?
**A:** Each RUN creates a new layer. Files deleted in a later layer still exist in earlier layers, consuming space. Cleanup must happen in the same RUN instruction to actually reduce layer size.

### Q6: What is the difference between Alpine and slim images?
**A:** Slim images use glibc (GNU C Library), ensuring maximum compatibility. Alpine uses musl libc, which is smaller but can cause compatibility issues with some software, especially Python native extensions.

### Q7: How do you scan images for vulnerabilities?
**A:** Use tools like Docker Scout (`docker scout cves`), Trivy, Grype, or Snyk. These scan image layers for known CVEs. Integrate into CI/CD to block deployments with critical vulnerabilities.

### Q8: What should never be in a Docker image?
**A:** Secrets (API keys, passwords, certificates), .git directories, development dependencies in production images, unnecessary tools (curl, wget in production), and cached files that won't be used.

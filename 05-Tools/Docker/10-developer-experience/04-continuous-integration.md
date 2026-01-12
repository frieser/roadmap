---
tags: ['docker', 'containers', 'ci-cd', 'devops', 'tools', 'roadmap']
---

# Continuous Integration

## Summary

Continuous Integration (CI/CD) pipelines automate the build, test, and deployment of containerized applications. Docker integrates with major CI/CD platforms (GitHub Actions, GitLab CI, CircleCI, Jenkins) through Docker actions, enabling consistent builds across environments. Key patterns include multi-platform builds, layer caching, security scanning, and deployment strategies.

## Detailed Explanation

### GitHub Actions

```yaml
# DOCKER BUILD AND PUSH
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
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ github.repository }}:latest,${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

```yaml
# MULTI-PLATFORM BUILDS
name: Multi-Platform Build

on:
  push:
    branches: [main]

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest]
        platform: [linux/amd64, linux/arm64]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: docker/setup-buildx-action@v3
        with:
          platforms: ${{ matrix.platform }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          platforms: ${{ matrix.platform }}
          push: true
          tags: myapp:latest,${{ github.sha }}
```

```yaml
# DOCKER IN DOCKER
name: Build in Docker

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: docker:24
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
      steps:
        - uses: actions/checkout@v4
        - name: Build and push
          uses: docker/build-push-action@v5
          with:
            context: .
            push: true
```
```

### GitLab CI

```yaml
# DOCKER BUILD AND TEST
image: docker:latest

services:
  docker:
    image: docker:24

variables:
  GIT_STRATEGY: clone

stages:
  - build
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

  - test
    stage: test
    services:
      - docker:dind
    script:
      - docker compose -f docker-compose.yml up -d
      - docker compose run --rm npm test
    artifacts:
      when: always
        paths:
          - coverage/

  - deploy
    stage: deploy
    script:
      - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      - docker-compose -f docker-compose.prod.yml up -d
    environment:
      - DEPLOYMENT: production
```

```yaml
# AUTO DEVOPS
image: docker:latest

stages:
  - build
  stage: build
  script:
    - docker build -t $CI_COMMIT_SHA .
  
  - deploy
  stage: deploy
  script:
    - docker-compose -f docker-compose.prod.yml up -d
  only:
      - main
  environment:
    - DEPLOYMENT: production
```

### CircleCI

```yaml
# DOCKER EXECUTOR
version: 2.1

executors:
  docker:
    machine:
      image: circleci/node:20
      # CircleCI's Docker executor
```

jobs:
  build:
    docker:
      - image: myapp:latest
        steps:
          - checkout
          - run: docker build -t myapp:$CIRCLE_SHA .
          - run: docker push myapp:$CIRCLE_SHA
```

### Jenkins

```yaml
# DOCKER AGENT
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        docker {
          image 'myapp:latest'
          args 'docker build -t myapp:$BUILD_NUMBER .'
          args 'docker push myapp:$BUILD_NUMBER'
        }
      }
    }
  }
}

# DEPLOYMENT
pipeline {
  agent any
  stages {
    stage('Deploy') {
      steps {
        ssh 'production-server' {
          git 'pull origin main'
          sh 'docker-compose -f docker-compose.prod.yml up -d'
        }
      }
    }
  }
}
```

### BuildKit Caching

```yaml
# GITHUB ACTIONS WITH CACHING
name: Build with Cache

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Set up BuildKit cache
        uses: actions/cache@v4
        with:
          path: ~/.cache/build
          key: ${{ runner.os }}-docker

      - name: Build with BuildKit
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: myapp:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            CACHE_FROM=type=gha
            CACHE_TO=type=gha,mode=max

# INLINE CACHE WITH BUILDKIT
# syntax=docker/dockerfile:1

# Cache npm packages
RUN --mount=type=cache,target=/root/.npm,id=npm_cache,sharing=locked \
    npm install

# Cache pip packages
RUN --mount=type=cache,target=/root/.cache/pip,id=pip_cache \
    pip install -r requirements.txt

# Cache Go modules
RUN --mount=type=cache,target=/go/pkg/mod,id=go_cache \
    go mod download
```

### Security Scanning

```yaml
# GITHUB ACTIONS - SCAN IMAGES
name: Scan Images

on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Build image
        uses: docker/build-push-action@v5
        with:
          push: false
          tags: myapp:latest

      - name: Run Trivy scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'
          format: 'sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: trivy-results
          path: ./trivy-results

# BLOCK VULNERABLE IMAGES
name: Scan and Block

on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'
          format: 'json'
          severity: 'CRITICAL,HIGH,UNKNOWN'
        
      - name: Block vulnerable images
        if: needs.scan && steps.scan.outputs.vulnerabilities.found == 'true'
        run: |
          echo "Critical vulnerabilities found. Blocking deployment."
          exit 1

# DOCKER SCOUT
name: Docker Scout

on:
  push:
    branches: [main]

jobs:
  scout:
    runs-on: ubuntu-latest
    steps:
      - name: Build image
        uses: docker/build-push-action@v5
        with:
          push: false
          tags: myapp:latest

      - name: Scan with Docker Scout
        run: |
          docker scout cves myapp:latest
          docker scout recommendations myapp:latest

      - name: Fail on CVEs
        if: steps.scan.outputs.critical-cves > 0
          run: |
            echo "Critical CVEs found. Failing build."
            exit 1
```

### Deployment Strategies

```yaml
# BLUE-GREEN DEPLOYMENT
version: '3.8'

services:
  # CURRENT (ACTIVE)
  app-green:
    image: myapp:latest-green
    deploy:
      mode: replicated
      replicas: 3

  # NEW VERSION (CANARY)
  app-blue:
    image: myapp:latest-blue
    deploy:
      mode: replicated
      replicas: 1

  # TRAFFIC SWITCHING
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    environment:
      - BLUE_PERCENT: 0
      - GREEN_PERCENT: 100

# ROLLING UPDATE (KUBERNETES)
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp-rollout

spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  updatePeriodSeconds: 30
  template:
        spec:
          containers:
            - name: myapp
              image: myapp:latest-green
```

### Registry Management

```yaml
# AUTOMATED IMAGE PROMOTION
# GitHub Actions

name: Image Promotion

on:
  push:
    tags:
      - v1.0.0

jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
      - name: Tag as stable
        run: |
          docker tag myapp:latest myregistry/myapp:v1.0.0
          docker push myregistry/myapp:v1.0.0

      - name: Tag as latest
        run: |
          docker tag myapp:latest myregistry/myapp:latest
          docker push myregistry/myapp:latest
```

```bash
# PROMOTE WITH DIGEST (IMMUTABLE)
# Pin digest for critical deployments
IMAGE_DIGEST=$(docker inspect myapp:latest --format '{{.Id}}')

docker tag myapp:${IMAGE_DIGEST} myregistry/myapp:stable
docker push myregistry/myapp:stable
# This tag always points to exact same image
```

## Interview Questions

### Q1: How do you use BuildKit caching in CI/CD pipelines?
**A:** BuildKit provides `--mount=type=cache` to cache package manager downloads (npm, pip, go modules) between builds. Use `--cache-from` and `--cache-to` to share cache across pipeline runs. This significantly speeds up builds by avoiding re-downloading dependencies.

### Q2: What is the difference between cache-from and cache-to?
**A:** `--cache-from` imports an existing cache from previous builds or a registry. `--cache-to` exports the current build cache to be used by future builds. Use both for optimal performance: import existing cache, export new cache.

### Q3: How do you implement blue-green deployments with Docker?
**A:** Run multiple versions of your application with different Docker image tags (e.g., `myapp:green`, `myapp:blue`). Use a traffic switch (like Nginx or a service mesh) to route a percentage of traffic to the new version. Gradually shift traffic and monitor for issues before switching fully.

### Q4: What is a rolling update strategy?
**A:** Rolling updates deploy new versions gradually, replacing old containers with new ones. In Kubernetes, specify `strategy.type: RollingUpdate` with `maxUnavailable` (how many pods can be down) and `updatePeriodSeconds` (time between updates). This ensures zero-downtime deployments.

### Q5: How do you scan Docker images for vulnerabilities in CI/CD?
**A:** Use tools like Trivy, Grype, or Docker Scout in CI/CD pipelines. Scan images before deployment and fail the pipeline if critical CVEs are found. Store scan results as artifacts for audit trails. Configure threshold policies (e.g., block on critical, warn on medium).

### Q6: How do you manage secrets in Docker CI/CD?
**A:** Never hardcode secrets in Dockerfiles or CI/CD configuration. Use encrypted secrets (GitHub Actions secrets, GitLab CI/CD variables, Kubernetes secrets). Use BuildKit `--mount=type=secret,id=mysecret,target=/app/secret` for build-time secrets. Rotate credentials regularly.

### Q7: What is multi-platform building with Docker BuildKit?
**A:** Use `docker buildx` which extends Docker with BuildKit to build for multiple platforms (linux/amd64, linux/arm64, windows/amd64) in a single command. Use `--platform` flag to specify targets. This enables cross-platform container images from a single build pipeline.

### Q8: How do you ensure reproducible builds in CI/CD?
**A:** Pin specific versions for all dependencies (Docker images, packages). Use `docker build --pull` to pull fresh base images. Use BuildKit cache for reproducibility. Tag images with git commit SHA for traceability: `myapp:${CI_COMMIT_SHA}`. Use digests for immutable references in critical deployments.

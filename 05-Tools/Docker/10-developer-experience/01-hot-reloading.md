---
tags: ['docker', 'containers', 'development', 'devops', 'tools', 'roadmap']
---

# Hot Reloading

## Summary

Hot reloading automatically restarts containerized applications when source code changes are detected. This dramatically improves development speed by eliminating the need to manually rebuild and restart containers after each code change. Approaches include volume mounts, Docker Compose watch, polling, and filesystem watchers in development mode.

## Detailed Explanation

### Volume Mount Hot Reload

```bash
# BIND MOUNT SOURCE CODE
# Changes in host directory trigger container restart

# Basic setup
docker run -d \
  --name myapp \
  -v $(pwd)/app:/app \
  -p 3000:3000 \
  myapp:latest

# Container must support hot reload
# - Detects file changes
# - Reloads application without restart
# - Or restarts automatically

# HOW HOT RELOAD WORKS
# 1. Edit file on host
vim app/main.py

# 2. Change reflected in container
# If app detects change and reloads = Done!

# Example: Go application with fsnotify
package main

import (
    "fmt"
    "log"
    "os"
    "os/signal"
    "path/filepath"
    "syscall"
)

func watchDirectory(dir string) error {
    watcher, err := fsnotify.NewWatcher()
    if err != nil {
        log.Fatal(err)
    }
    defer watcher.Close()

    if err := watcher.Add(dir); err != nil {
        log.Fatal(err)
    }
    
    for {
        select {
        case <-watcher.Events:
            case event := <-watcher.Events:
                log.Printf("Modified: %s", event.Name)
                // Send reload signal to app
                syscall.Kill(syscall.SIGUSR2, syscall.SIGINT)
        }
    }
}

func main() {
    watchDirectory("./app")
}
```

```yaml
# Node.js with nodemon (in container)
# docker-compose.yml for development
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src  # Bind mount for hot reload
      - /app/node_modules  # Volume for dependencies
    command: npm run dev
```
```

### Docker Compose Watch

```bash
# DOCKER COMPOSE WATCH (Modern approach)
docker compose watch

# AUTOMATICALLY RELOADS WHEN FILES CHANGE
# - No manual container restart needed
# - Works with all major languages/frameworks

# SETUP REQUIREMENTS
# - Docker Compose v2.20+
# - Containers with hot reload support
# - Local source code mounted

# MANUAL TRIGGER
docker compose watch --build  # Rebuilds when files change
docker compose watch  # Watches files only, no rebuild

# LANGUAGE-SPECIFIC WATCHERS
# - Go: Embeds watcher in container (most common)
# - Node: nodemon detects changes
# - Python: Watchdog or custom script
# - JavaScript: nodemon or PM2
```

```yaml
# DOCKER-COMPOSE.YML WITH WATCH
version: '3.8'

services:
  app:
    build: .
    volumes:
      - ./src:/app/src
      - ./package.json:/app/package.json
    develop:
      watch:
        - path: ./src
        - target: /app/src
        - action: sync  # Sync changes into container

  python-app:
    build: ./python-app
    volumes:
      - ./python-app:/app
    develop:
      watch:
        - path: ./python-app
        - target: /app
        - action: rebuild  # Rebuild on Python changes
```
```

### Docker Init and Development Tools

```bash
# DOCKER INIT (Newer approach)
docker init
# Creates docker-compose.yml, .gitignore, .dockerignore
# Interactive wizard for setup

# SETUP DEVELOPMENT ENVIRONMENT
docker init
# Follow prompts:
# - Project name
# - Services to include
# - Language (Python, Go, Node, etc.)
# - Database integration

# RUN INITIALIZED PROJECT
cd my-docker-project
docker compose up
```

```yaml
# docker-init GENERATED docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "80:80"
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
```

### Development Container Images

```yaml
# IMAGES WITH DEVELOPMENT TOOLS

# NODE.JS WITH DEV TOOLS
FROM node:20-alpine

# Install development tools
RUN npm install -g \
  nodemon@latest \
  eslint@latest \
  @types/node@latest \
  typescript@latest \
  --save-prod=false

# Copy application
WORKDIR /app
COPY . .

# Use nodemon for hot reload
CMD ["npx", "nodemon"]
```

```dockerfile
# GO WITH DELVE
FROM golang:1.21-alpine AS builder

# Install Delve (Go debugger)
RUN apk add --no-cache git && \
    git clone --depth=1 https://github.com/go-delve/delve.git /delve && \
    cd delve && \
    go install -v . github.com/go-delve/delve

# Build application
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -gcflags="all=-N -l" -o /app/app .

# Final stage
FROM golang:1.21-alpine

# Install Delve in final image
COPY --from=builder /delve/delve /usr/local/bin/delve

# Set up for debugging
WORKDIR /app
COPY --from=builder /app/app .
ENV DELVE_APP=/app/app
ENV DELVE_LISTEN=2345

# Run with Delve
CMD ["/usr/local/bin/delve", "listen=:2345", "--headless", "--api-version=2", "exec", "/app/app"]
```

### Filesystem Watchers

```yaml
# POLLING WATCHERS (Simpler, less efficient)
watch_strategies:
  polling:
    implementation: "Check file modification times"
    pros: ["Simple", "Works everywhere"]
    cons: ["High CPU usage", "Delayed detection"]
    languages: ["Python", "Go"]
  
  inotify_watchers:
    implementation: "Use OS filesystem events"
    pros: ["Instant detection", "Low overhead"]
    cons: ["Linux only", "Limit on number of watches"]
    languages: ["Go", "Rust", "Node.js"]
  
  custom_watchers:
    implementation: "Application-level logic"
    pros: ["Exact control", "Framework integration"]
    cons: ["More complex to implement"]

# EXAMPLE: Node.js with chokidar
package.json:
{
  "devDependencies": {
    "chokidar": "^3.5.1",
    "nodemon": "^3.1.0"
  },
  "scripts": {
    "dev": "nodemon --watch src app.js",
    "watch": "chokidar src -c 'node server.js'"
  }
}

# Docker Compose with chokidar
services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev
```

### Language-Specific Hot Reload

```bash
# GO: Embed watcher in container
# Most common approach
# fsnotify package in Go stdlib

# PYTHON: Use file system polling or watchdog
# Simpler, cross-platform
# watchdog package works well

# NODE.JS: nodemon or PM2
# nodemon: Standard choice
# PM2: Alternative with better performance

# RUBY: Guard-listen or rerun
# Guard: Automatic restart on file changes

# JAVA: Spring Boot DevTools
# Automatic restart with spring-boot-devtools
# JVM hot swap enabled

# DOTNET: dotnet watch
# Monitors file system for .NET projects
# Built-in tool from .NET SDK

# PHP: Laravel Mix or Watchman
# Framework-integrated hot reload
```

```yaml
# LANGUAGE-SPECIFIC docker-compose.yml

# Python with watchdog
version: '3.8'

services:
  python-app:
    build: ./python-app
    command: python -m watchdog -w /app -e FLASK_ENV=development app:app
    volumes:
      - .:/app

# Node.js with nodemon
version: '3.8'

services:
  node-app:
    build: ./node-app
    command: npx nodemon src/app.js
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development

# Go with Air (hot reload tool)
version: '3.8'

services:
  go-app:
    build: ./go-app
    command: air
    volumes:
      - .:/app
    environment:
      - GO_ENV=development
```

### Remote Debugging

```bash
# DOCKER FORWARDED PORTS
docker run -d \
  -p 4000:4000 \
  -p 2345:2345 \
  -e DEBUG=true \
  --name myapp \
  myapp:latest

# Port 4000: Application
# Port 2345: Delve debugger
# Connect debugger to container: dlv connect :2345

# SSH INTO CONTAINER FOR DEBUG
docker exec -it myapp bash

# SET BREAKPOINTS IN DEBUGGER
dlv breakpoint main.go:45

# USE DOCKER DEBUG MODE
docker run -d \
  -p 2345:2345 \
  -e DELVE_DEBUG=true \
  myapp:latest
```

### CI/CD Hot Reload Considerations

```yaml
# HOT RELOAD NOT APPROPRIATE IN CI/CD
# CI: Build only, run unit tests
# CD: Build and deploy new image, don't watch files

# SEPARATE CONCERNS
# Application handles hot reload in production
# Separate worker processes for background tasks
# Don't rebuild containers continuously in production

# ENVIRONMENT-SPECIFIC BUILD
development:
  watch: true  # Enable hot reload tools
  target: development  # Build for dev environment

production:
  watch: false  # Disable hot reload
  target: production  # Production-optimized build

# GITLAB CI EXAMPLE
build_dev:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:dev .
    - docker push $CI_REGISTRY_IMAGE:dev

build_prod:
  stage: build
  script:
    - docker build --no-cache -t $CI_REGISTRY_IMAGE:prod .
    - docker push $CI_REGISTRY_IMAGE:prod

deploy_dev:
  stage: deploy
  script:
    - docker compose -f docker-compose.dev.yml up -d
  watch: true

deploy_prod:
  stage: deploy
  script:
    - docker compose -f docker-compose.prod.yml up -d
```

## Interview Questions

### Q1: What is hot reloading in the context of Docker?
**A:** Hot reloading automatically restarts containerized applications when source code changes are detected, eliminating the need to manually rebuild and restart containers after each code change during development. Approaches include volume mounts, Docker Compose watch, and file system watchers.

### Q2: How do volume mounts enable hot reload?
**A:** By mounting your local source code directory into the container (`-v $(pwd)/app:/app`), changes made on the host are immediately visible inside the container. Applications with hot reload support (like Go with fsnotify, Node.js with nodemon) can detect these changes and reload automatically.

### Q3: What is Docker Compose watch?
**A:** `docker compose watch` is a modern feature that automatically rebuilds or restarts containers when files change. It's available in Docker Compose v2.20+ and works with language-specific watchers embedded in many container images. It's more reliable than manual volume mounts for complex applications.

### Q4: How does `docker init` help with development?
**A:** `docker init` is an interactive tool that creates initial Docker configuration files (docker-compose.yml, .dockerignore, .gitignore) based on prompts. It detects your project type, language, and can set up common development services like databases automatically.

### Q5: What is the difference between polling and inotify for file watching?
**A:** Polling checks file modification times periodically - simpler to implement but higher CPU usage. Inotify (fsnotify in Go) uses kernel-level events - instant detection with minimal overhead. However, inotify only works on Linux and has limits on the number of watches.

### Q6: How do you debug a Docker container?
**A:** Use language-specific debuggers with remote debugging support. For Go, use Delve with `dlv connect <host-port>:<container-port>`. For Node.js, use Chrome DevTools with `--inspect` flag. Or use `docker exec` to run debugging tools inside the container.

### Q7: When is hot reloading not appropriate for production?
**A:** Hot reloading is not recommended for production due to performance overhead, reliability concerns, and potential security risks from hot code swaps. In production, build and deploy immutable container images, restart containers on configuration changes, and use orchestration tools like Kubernetes for rolling deployments.

### Q8: What is a file system watcher vs application-level hot reload?
**A:** File system watchers (like nodemon, chokidar, fsnotify) detect file system changes and trigger application reload. Application-level hot reload (like Next.js Fast Refresh, Vue HMR) is built into frameworks and typically faster but more complex to implement. Choose based on your technology stack and requirements.

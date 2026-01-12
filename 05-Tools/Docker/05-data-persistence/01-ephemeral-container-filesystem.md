---
tags: ['docker', 'containers', 'storage', 'devops', 'tools', 'roadmap']
---

# Ephemeral Container Filesystem

## Summary

Container filesystems are ephemeral by default - all changes made inside a running container are lost when the container is removed. Each container gets a thin writable layer on top of the image's read-only layers. This layer exists only for the container's lifetime. Understanding this ephemeral nature is crucial for designing stateful applications and knowing when to use volumes or bind mounts for persistent data.

## Detailed Explanation

### How Container Storage Works

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTAINER                                 │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         Writable Container Layer                      │  │
│  │    (Unique per container, ephemeral)                  │  │
│  │    - New files created here                           │  │
│  │    - Modified files copied here (copy-on-write)       │  │
│  │    - Deleted when container is removed                │  │
│  └───────────────────────────────────────────────────────┘  │
│                          │                                   │
│                    Union Mount                               │
│                          │                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         Read-Only Image Layers                        │  │
│  │    Layer 3: Application code                          │  │
│  │    Layer 2: Dependencies                              │  │
│  │    Layer 1: Base OS filesystem                        │  │
│  │    (Shared across all containers using this image)    │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Demonstrating Ephemeral Storage

```bash
# Create a file inside a container
docker run -d --name mycontainer ubuntu:22.04 sleep 1000
docker exec mycontainer bash -c 'echo "Hello" > /data.txt'
docker exec mycontainer cat /data.txt
# Hello

# Stop container - file still exists
docker stop mycontainer
docker start mycontainer
docker exec mycontainer cat /data.txt
# Hello

# Remove container - file is GONE
docker rm -f mycontainer
docker run --name mycontainer ubuntu:22.04 cat /data.txt
# cat: /data.txt: No such file or directory
```

### Copy-on-Write Behavior

```bash
# When you modify a file from the image:
# 1. File is copied from image layer to writable layer
# 2. Modification applied in writable layer
# 3. Original in image layer unchanged

# Example:
docker run -d --name nginx-container nginx:alpine
docker exec nginx-container cat /etc/nginx/nginx.conf  # Original
docker exec nginx-container sh -c 'echo "# Modified" >> /etc/nginx/nginx.conf'
docker exec nginx-container cat /etc/nginx/nginx.conf  # Modified

# But the image is unchanged:
docker run --rm nginx:alpine cat /etc/nginx/nginx.conf  # Original

# The modification only exists in nginx-container's writable layer
docker rm -f nginx-container
# Modification is now gone forever
```

### Viewing Container Layer Size

```bash
# See how much data is in the writable layer
docker ps -s
# CONTAINER ID   IMAGE   ...   SIZE
# abc123         nginx   ...   5B (virtual 187MB)
# def456         nginx   ...   0B (virtual 187MB)

# SIZE = writable layer size
# virtual = total including shared image layers

# Inspect container for detailed storage info
docker inspect mycontainer --format '{{.GraphDriver.Data}}'

# View layer location (overlay2)
docker inspect mycontainer --format '{{.GraphDriver.Data.UpperDir}}'
# /var/lib/docker/overlay2/<id>/diff

# List files in writable layer
sudo ls /var/lib/docker/overlay2/<container-id>/diff/
```

### When Ephemeral is Good

```yaml
# Ephemeral storage is APPROPRIATE for:

stateless_applications:
  - Web servers serving static content
  - API servers (state in database)
  - Compute workers (state in message queue)

temporary_data:
  - Cache files
  - Session data (if okay to lose)
  - Compiled artifacts during build
  - Test data

benefits:
  - Clean state on each container restart
  - No data accumulation/bloat
  - Easy horizontal scaling
  - Simplified container lifecycle
  - Reproducible environments

# Example: stateless web server
docker run -d --name web nginx
# Restart = fresh container
docker rm -f web && docker run -d --name web nginx
```

### When Ephemeral is a Problem

```yaml
# Ephemeral storage is PROBLEMATIC for:

databases:
  - PostgreSQL, MySQL, MongoDB
  - Losing data on container restart = disaster

user_uploads:
  - Uploaded files would disappear
  - Media, documents, attachments

logs_and_audit:
  - Application logs
  - Security audit trails
  - Compliance records

configuration:
  - Dynamic configuration changes
  - Certificates (if rotated in container)

# SOLUTION: Use volumes or bind mounts
docker run -d \
  --name postgres \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15
# Now data persists even if container is removed
```

### tmpfs - In-Memory Ephemeral

```bash
# tmpfs mounts use host memory, not disk
# Data lost on container stop (not just remove)

docker run -d \
  --name myapp \
  --tmpfs /app/temp:size=100m \
  myapp:latest

# Or with --mount syntax
docker run -d \
  --mount type=tmpfs,destination=/app/temp,tmpfs-size=100m \
  myapp:latest

# Use cases:
# - Sensitive data that shouldn't touch disk
# - High-speed temporary files
# - Secrets that should disappear

# Check tmpfs mounts
docker inspect myapp --format '{{json .Mounts}}'
```

### Container Filesystem Best Practices

```yaml
# 1. DESIGN FOR EPHEMERAL
# Assume container can be replaced anytime
# Don't store important data in container filesystem

# 2. IDENTIFY STATEFUL DATA
# What data must persist?
# Databases, uploads, logs, config

# 3. USE APPROPRIATE STORAGE
stateful_data: "Named volumes"
host_files: "Bind mounts"
sensitive_temp: "tmpfs"
everything_else: "Container filesystem (ephemeral)"

# 4. SEPARATE CONCERNS
docker run -d \
  --name myapp \
  -v app_data:/app/data \           # Persistent data
  -v ./config:/app/config:ro \      # Configuration
  --tmpfs /app/temp \               # Temp files
  myapp:latest
  # /app/code is ephemeral (from image)

# 5. TWELVE-FACTOR APP APPROACH
# Store config in environment variables
# Keep state in backing services (database)
# Treat containers as disposable
```

### Commit (Anti-Pattern)

```bash
# docker commit creates image from container state
# Includes writable layer changes

docker run -it --name modified ubuntu:22.04 bash
# Inside: apt-get update && apt-get install -y curl
exit

docker commit modified my-ubuntu-with-curl:v1
docker images
# REPOSITORY            TAG   IMAGE ID
# my-ubuntu-with-curl   v1    abc123

# WHY THIS IS BAD:
# - Not reproducible (no Dockerfile)
# - Unknown changes in the layer
# - Harder to maintain and audit
# - Can't optimize layer caching

# USE DOCKERFILE INSTEAD:
# FROM ubuntu:22.04
# RUN apt-get update && apt-get install -y curl
```

### Inspecting Container Changes

```bash
# See what changed in container filesystem
docker diff mycontainer
# A /app/new-file.txt      (Added)
# C /etc/nginx             (Changed)
# D /tmp/old-file.txt      (Deleted)

# Export container filesystem
docker export mycontainer > container-fs.tar
tar -tf container-fs.tar

# Copy files from container
docker cp mycontainer:/app/data.txt ./data.txt

# Copy files to container (for debugging)
docker cp ./config.json mycontainer:/app/config.json
```

## Interview Questions

### Q1: What happens to data inside a container when you remove it?
**A:** Data in the container's writable layer is permanently deleted when the container is removed. Only data in volumes or bind mounts persists. This is why databases and other stateful applications must use volumes.

### Q2: What is the difference between stopping and removing a container regarding data?
**A:** Stopping a container pauses it but preserves the writable layer - data survives and is available when restarted. Removing a container deletes the writable layer and all data in it permanently.

### Q3: What is copy-on-write in the context of containers?
**A:** When a container modifies a file from the image layers, the file is first copied to the writable layer, then modified there. The original in the image layer remains unchanged. This allows image layers to be shared and immutable.

### Q4: When is ephemeral storage appropriate?
**A:** For stateless applications (web servers, API services), temporary data (caches, build artifacts), and when clean state on restart is desired. Ephemeral storage is fine when the application stores persistent state externally (database, object storage).

### Q5: What is tmpfs and when would you use it?
**A:** tmpfs is an in-memory filesystem. Data is lost when the container stops (not just removed). Use for sensitive data that shouldn't touch disk, high-speed temporary files, or secrets that must disappear immediately.

### Q6: Why is `docker commit` considered an anti-pattern?
**A:** It creates images from running container state, which is not reproducible, hard to audit, and bypasses Dockerfile best practices. Use Dockerfiles instead - they're version-controlled, reproducible, and support layer caching.

### Q7: How do you see what files changed inside a running container?
**A:** Use `docker diff <container>` which shows Added (A), Changed (C), and Deleted (D) files compared to the image. This helps understand what the container has modified.

### Q8: What is the Twelve-Factor approach to container data?
**A:** Treat containers as disposable, store configuration in environment variables, and keep all persistent state in backing services (databases, caches, object storage). The container itself should be stateless and replaceable.

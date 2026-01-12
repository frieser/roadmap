---
tags: ['docker', 'containers', 'linux', 'devops', 'tools', 'roadmap']
---

# Union Filesystems

## Summary

Union filesystems (UnionFS) allow multiple filesystem layers to be stacked and presented as a single unified view. Docker uses union filesystems (like OverlayFS, AUFS) to create efficient container images where layers are shared and reused. Each image layer is read-only, and containers add a writable layer on top. This enables fast image distribution, efficient disk usage, and copy-on-write semantics for container storage.

## Detailed Explanation

### How Union Filesystems Work

```
┌──────────────────────────────────────────────────────────────┐
│                  UNIFIED VIEW (Container)                     │
│    /                                                          │
│    ├── bin/       (from base layer)                          │
│    ├── etc/       (merged from multiple layers)              │
│    │   └── nginx.conf  (from app layer, overrides base)     │
│    ├── app/       (from app layer)                           │
│    └── logs/      (from container writable layer)            │
└──────────────────────────────────────────────────────────────┘
                            ▲
                            │ union mount
     ┌──────────────────────┼───────────────────────┐
     │                      │                       │
┌────┴─────┐         ┌──────┴──────┐         ┌──────┴──────┐
│ Writable │         │ App Layer   │         │ Base Layer  │
│ Layer    │         │ (read-only) │         │ (read-only) │
│          │         │             │         │             │
│ logs/    │         │ app/        │         │ bin/        │
│ new.txt  │         │ etc/nginx.. │         │ etc/        │
└──────────┘         └─────────────┘         │ lib/        │
(top)                                        └─────────────┘
                                             (bottom)

Files from higher layers override lower layers
New files go to writable layer
```

### OverlayFS (Modern Default)

```bash
# OverlayFS is Docker's default storage driver
# Merged view of multiple directories

# OverlayFS structure:
# lowerdir: Read-only image layers (can be multiple)
# upperdir: Writable container layer
# workdir: Scratch space for overlay operations
# merged: Unified view presented to container

# Check Docker storage driver
docker info | grep -i storage
# Storage Driver: overlay2

# View overlay mount for a container
docker run -d --name web nginx
cat /proc/$(docker inspect web --format '{{.State.Pid}}')/mountinfo | grep overlay
# Shows overlay mount details

# Manual overlay example
mkdir lower upper work merged

# Create files in lower
echo "base file" > lower/base.txt

# Mount overlay
sudo mount -t overlay overlay \
  -o lowerdir=lower,upperdir=upper,workdir=work \
  merged/

# See unified view
ls merged/  # base.txt

# Create new file - goes to upper
echo "new file" > merged/new.txt
ls upper/   # new.txt (written to upper, not lower)

# Modify base file - copy-up to upper
echo "modified" > merged/base.txt
ls upper/   # base.txt (copied up and modified)
ls lower/   # base.txt (unchanged)
```

### Copy-on-Write (CoW)

```bash
# Copy-on-Write: Files copied to writable layer only when modified

# Original state:
# lower/app.py (100MB, read-only)
# upper/ (empty)

# Read file - no copy
cat merged/app.py
# Reads directly from lower layer
# upper/ still empty

# Modify file - triggers copy
echo "# comment" >> merged/app.py
# 1. File copied from lower to upper
# 2. Modification applied in upper
# ls upper/  # app.py (100MB copy now exists)

# Benefits of CoW:
# - Multiple containers share read-only layers
# - Only modified files consume additional space
# - Fast container creation (no full copy)

# Downsides:
# - First write to large file is slow (full copy)
# - Repeated writes accumulate in upper layer
# - Not ideal for databases (use volumes instead)
```

### Docker Image Layers

```dockerfile
# Each Dockerfile instruction creates a layer
FROM ubuntu:22.04       # Layer 1: Base OS (~77MB)
RUN apt-get update && \
    apt-get install -y python3  # Layer 2: Python (~50MB)
COPY requirements.txt . # Layer 3: requirements.txt (~1KB)  
RUN pip install -r requirements.txt  # Layer 4: Dependencies (~20MB)
COPY app/ /app/         # Layer 5: Application code (~500KB)
CMD ["python3", "/app/main.py"]  # No layer (metadata only)
```

```bash
# View image layers
docker history myapp:latest
# IMAGE          CREATED        CREATED BY                            SIZE
# abc123         1 minute ago   CMD ["python3" "/app/main.py"]        0B
# def456         1 minute ago   COPY app/ /app/                       500kB
# ghi789         2 minutes ago  RUN pip install -r requirements.txt   20MB
# jkl012         2 minutes ago  COPY requirements.txt .               1kB
# mno345         3 minutes ago  RUN apt-get update && apt-get...     50MB
# ubuntu:22.04   2 weeks ago    ...                                   77MB

# Layers are shared between images
docker images
# REPOSITORY  TAG     IMAGE ID   SIZE
# myapp       v1      abc123     147MB
# myapp       v2      def456     148MB  # Shares base layers with v1

# View layer storage
ls /var/lib/docker/overlay2/
# Contains layer directories

docker inspect myapp:latest --format '{{json .RootFS.Layers}}'
# Shows layer digest list
```

### Layer Caching

```dockerfile
# Order matters for layer caching!

# BAD - Code changes invalidate dependency layer
FROM python:3.11
COPY . /app                    # Changes frequently
RUN pip install -r /app/requirements.txt  # Rebuilds every time!

# GOOD - Dependencies cached separately
FROM python:3.11
COPY requirements.txt /app/    # Changes rarely
RUN pip install -r /app/requirements.txt  # Cached!
COPY . /app                    # Only this layer rebuilds on code change
```

```bash
# Build with cache
docker build -t myapp:v1 .
# Step 2/4: COPY requirements.txt
#  ---> Using cache
# Step 3/4: RUN pip install
#  ---> Using cache       <- Cached because requirements.txt unchanged
# Step 4/4: COPY . /app
#  ---> abc123def         <- Only this rebuilds

# Force rebuild without cache
docker build --no-cache -t myapp:v1 .
```

### Storage Drivers

```bash
# Docker supports multiple storage drivers:

# OVERLAY2 (recommended, default)
# - Modern, efficient
# - Good performance
# - Requires kernel 4.0+

# AUFS (legacy)
# - Original Docker driver
# - Not in mainline kernel
# - Debian/Ubuntu only

# DEVICEMAPPER
# - Block-level storage
# - Used on older RHEL/CentOS
# - Direct-lvm mode for production

# BTRFS / ZFS
# - Advanced filesystems
# - Built-in snapshotting
# - Requires BTRFS/ZFS formatted storage

# Check current driver
docker info | grep "Storage Driver"

# Configure in /etc/docker/daemon.json
{
  "storage-driver": "overlay2"
}
```

### Container Writable Layer

```bash
# Each container has its own writable layer

# Create two containers from same image
docker run -d --name web1 nginx
docker run -d --name web2 nginx

# Modify files in web1
docker exec web1 sh -c 'echo "modified" > /etc/nginx/nginx.conf'

# web2 is unaffected (different writable layer)
docker exec web2 cat /etc/nginx/nginx.conf
# Original content

# View container size
docker ps -s
# CONTAINER  IMAGE  ...  SIZE
# web1       nginx  ...  5B (virtual 187MB)
# web2       nginx  ...  0B (virtual 187MB)
# 5B = writable layer size
# 187MB = total with shared layers

# Writable layer is ephemeral!
docker rm web1
# All changes in writable layer are lost
```

### Inspecting Layers

```bash
# View image layer details
docker inspect nginx --format '{{json .RootFS.Layers}}' | jq
# [
#   "sha256:abc123...",  # Layer 1
#   "sha256:def456...",  # Layer 2
#   ...
# ]

# Explore layer content
# Find layer directory
ls /var/lib/docker/overlay2/

# Layer diff contains that layer's files
ls /var/lib/docker/overlay2/<layer-id>/diff/

# Export image as tar to examine
docker save nginx -o nginx.tar
tar -tf nginx.tar
# manifest.json
# abc123.../layer.tar  # Each layer as tar
# def456.../layer.tar

# Examine specific layer
tar -xf nginx.tar abc123.../layer.tar
tar -tf abc123.../layer.tar
```

### Best Practices

```dockerfile
# 1. MINIMIZE LAYERS
# Combine RUN commands
RUN apt-get update && \
    apt-get install -y \
      package1 \
      package2 && \
    rm -rf /var/lib/apt/lists/*

# 2. ORDER BY CHANGE FREQUENCY
# Least frequently changed → Most frequently changed
COPY go.mod go.sum ./     # Dependencies (stable)
RUN go mod download
COPY . .                   # Source code (changes often)

# 3. USE .dockerignore
# Prevent unnecessary files from being copied
# .dockerignore:
.git
node_modules
*.log
.env

# 4. CLEAN UP IN SAME LAYER
# Bad - leaves cache in layer
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*  # Cache still in previous layers!

# Good - cleanup in same layer
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# 5. USE MULTI-STAGE BUILDS
# Build stage layers don't end up in final image
FROM golang:1.21 AS builder
COPY . .
RUN go build -o app

FROM alpine:3.18
COPY --from=builder /app /app  # Only copy the binary
```

## Interview Questions

### Q1: What is a union filesystem?
**A:** A union filesystem merges multiple directory trees (layers) into a single unified view. Upper layers can override or add to lower layers. Docker uses this to stack read-only image layers with a writable container layer.

### Q2: What is copy-on-write?
**A:** Copy-on-write (CoW) means files are only copied to the writable layer when modified. Reading uses the original layer; writing triggers a full copy to the upper layer. This enables sharing layers between containers while allowing modifications.

### Q3: What storage driver does Docker use by default?
**A:** OverlayFS (overlay2) is the default on modern Linux systems. It provides efficient layer management with good performance. Older systems might use AUFS or devicemapper.

### Q4: Why is the order of Dockerfile instructions important for caching?
**A:** Docker caches layers and reuses them if the instruction and context haven't changed. If an early layer changes, all subsequent layers must rebuild. Order instructions from least to most frequently changing to maximize cache hits.

### Q5: What happens to container changes when the container is removed?
**A:** Changes in the container's writable layer are lost when the container is removed. The writable layer is ephemeral and tied to the container's lifecycle. Use volumes for persistent data.

### Q6: How are layers shared between containers?
**A:** Image layers are read-only and stored once on disk. Multiple containers from the same image share these layers (via overlay mounts). Each container only has its own unique writable layer.

### Q7: Why is it important to clean up in the same RUN instruction?
**A:** Each RUN creates a layer that captures the filesystem state. Deleting files in a later RUN hides them but doesn't reduce image size - the files exist in the earlier layer. Cleanup must happen in the same instruction.

### Q8: What is the disadvantage of CoW for database workloads?
**A:** CoW causes write amplification - first write to any file copies the entire file. Databases with many small writes perform poorly. Use Docker volumes (bypass union filesystem) for database data directories.

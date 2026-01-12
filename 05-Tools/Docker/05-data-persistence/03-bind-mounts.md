---
tags: ['docker', 'containers', 'storage', 'devops', 'tools', 'roadmap']
---

# Bind Mounts

## Summary

Bind mounts link a specific file or directory on the host machine directly into a container. Unlike volumes which are managed by Docker, bind mounts depend on the host's directory structure. They're ideal for development workflows (live code reloading), configuration files, and accessing host-specific paths. Changes are reflected bidirectionally - modifications in either the host or container are immediately visible to both.

## Detailed Explanation

### How Bind Mounts Work

```
HOST FILESYSTEM:
/home/user/project/
├── src/
│   └── app.py
├── config/
│   └── settings.json
└── data/

CONTAINER VIEW (with bind mount):
/app/
├── src/           <- Same files as host's /home/user/project/src/
│   └── app.py
├── config/        <- Same files as host's /home/user/project/config/
│   └── settings.json
└── data/          <- Same files as host's /home/user/project/data/

# Changes in either location are immediately visible in both
# Edit app.py on host → Container sees change immediately
# Create file in container → Appears on host immediately
```

### Creating Bind Mounts

```bash
# Short syntax (-v)
docker run -d \
  -v /host/path:/container/path \
  myapp

# Long syntax (--mount) - more explicit, recommended
docker run -d \
  --mount type=bind,source=/host/path,target=/container/path \
  myapp

# Current directory
docker run -d \
  -v $(pwd):/app \
  myapp

# Windows path
docker run -d \
  -v "C:\Users\name\project:/app" \
  myapp

# With options
docker run -d \
  -v /host/path:/container/path:ro \
  myapp

docker run -d \
  --mount type=bind,source=/host/path,target=/container/path,readonly \
  myapp
```

### Bind Mount Options

```bash
# READ-ONLY mount (container can't modify)
docker run -v $(pwd)/config:/app/config:ro myapp
docker run --mount type=bind,source=$(pwd)/config,target=/app/config,readonly myapp

# CONSISTENCY options (macOS only)
# consistent - Full consistency (default, slow)
# cached - Host authoritative, container has delayed view
# delegated - Container authoritative, host has delayed view

docker run -v $(pwd)/src:/app/src:cached myapp     # Dev: faster reads
docker run -v $(pwd)/output:/app/output:delegated myapp  # Build: faster writes

# PROPAGATION (Linux only)
# rprivate - No propagation (default)
# private - Sub-mounts not visible to host
# rshared/rslave - Propagate mount events

docker run -v /host/path:/container/path:rshared myapp
```

### Development Workflow

```yaml
# docker-compose.yml for development
version: '3.8'

services:
  app:
    build: .
    volumes:
      # Bind mount source code for hot reload
      - ./src:/app/src
      - ./config:/app/config:ro
      
      # BUT use volume for dependencies (not bind mount)
      - node_modules:/app/node_modules
    
    environment:
      - NODE_ENV=development
    
    command: npm run dev  # Hot-reloading dev server

volumes:
  node_modules:
```

```bash
# Python development example
docker run -it --rm \
  -v $(pwd)/app:/app \
  -w /app \
  python:3.11 \
  python -m flask run --reload --host=0.0.0.0

# File changes on host → Flask auto-reloads

# Node.js development
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:20 \
  npx nodemon app.js

# Go development with air
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 8080:8080 \
  cosmtrek/air
```

### Single File Bind Mounts

```bash
# Mount single file
docker run -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# Mount configuration file
docker run \
  -v $(pwd)/my-postgres.conf:/etc/postgresql/postgresql.conf:ro \
  postgres:15

# Mount credentials (carefully!)
docker run \
  -v ~/.aws/credentials:/root/.aws/credentials:ro \
  amazon/aws-cli s3 ls

# IMPORTANT: If file doesn't exist, Docker creates a directory!
# Wrong:
docker run -v $(pwd)/config.json:/app/config.json nginx
# If config.json doesn't exist, /app/config.json becomes a directory

# Solution: Ensure file exists before mounting
touch config.json
docker run -v $(pwd)/config.json:/app/config.json nginx
```

### Performance Considerations

```yaml
# BIND MOUNT PERFORMANCE ISSUES (Docker Desktop)
# On macOS/Windows, bind mounts go through a VM layer
# This adds significant overhead for many small files

performance_tips:
  # 1. Use volumes for dependencies
  # Bad: Mount entire project including node_modules
  - ./:/app
  
  # Good: Bind mount source, volume for deps
  - ./src:/app/src
  - node_modules:/app/node_modules

  # 2. Use cached/delegated on macOS
  - ./src:/app/src:cached
  
  # 3. Minimize mounted files
  # Don't mount .git, node_modules, __pycache__
  # Use .dockerignore for build context
  
  # 4. Consider docker-sync (macOS, third-party)
  # Keeps host and container in sync with better perf

# LINUX: Bind mounts are native, no performance penalty
```

### Ownership and Permissions

```bash
# Container often runs as root, creating files as root on host
docker run -v $(pwd)/data:/data ubuntu:22.04 touch /data/file.txt
ls -la data/
# -rw-r--r-- 1 root root 0 file.txt  <- Owned by root!

# Solution 1: Run container as current user
docker run \
  --user $(id -u):$(id -g) \
  -v $(pwd)/data:/data \
  ubuntu:22.04 touch /data/file.txt
ls -la data/
# -rw-r--r-- 1 user user 0 file.txt  <- Owned by you

# Solution 2: Match container user to host user
# In Dockerfile:
# ARG UID=1000
# ARG GID=1000
# RUN groupadd -g $GID appgroup && useradd -u $UID -g $GID appuser
# USER appuser

# Solution 3: Fix permissions after
docker run -v $(pwd)/output:/output myapp build
sudo chown -R $(id -u):$(id -g) output/

# Docker Compose with user
services:
  app:
    image: myapp
    user: "${UID:-1000}:${GID:-1000}"
    volumes:
      - ./data:/app/data
```

### Bind Mounts vs Volumes

```yaml
bind_mounts:
  stored: "Host filesystem, any path"
  managed: "By you (the user)"
  host_access: "Direct - edit files with any editor"
  portability: "Path must exist on host"
  performance: "Variable (slow on Docker Desktop)"
  backup: "Use host backup tools"
  
  best_for:
    - Development (live code reload)
    - Configuration files
    - Build outputs
    - Host file access

volumes:
  stored: "Docker-managed (/var/lib/docker/volumes/)"
  managed: "By Docker"
  host_access: "Via Docker commands or root"
  portability: "Works on any Docker host"
  performance: "Consistent, optimized"
  backup: "Use Docker volume commands"
  
  best_for:
    - Persistent data (databases)
    - Shared data between containers
    - Production workloads
    - When you don't need host access
```

### Docker Compose Examples

```yaml
version: '3.8'

services:
  # Development setup
  web:
    build: .
    volumes:
      # Source code (bind mount for hot reload)
      - ./src:/app/src
      - ./public:/app/public
      
      # Config (read-only bind mount)
      - ./config/app.json:/app/config/app.json:ro
      
      # Dependencies (named volume, not bind mount)
      - node_modules:/app/node_modules
      
      # Build output (bind mount to access from host)
      - ./dist:/app/dist

  # Database with volume (not bind mount)
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
      # Init scripts (bind mount, read-only)
      - ./init-db:/docker-entrypoint-initdb.d:ro

volumes:
  node_modules:
  postgres_data:
```

### Troubleshooting

```bash
# File not found / is a directory
# Docker created directory because file didn't exist
rm -rf /path/that/became/directory
touch /path/to/file
docker run -v /path/to/file:/container/file myapp

# Permission denied
docker run --user $(id -u):$(id -g) -v $(pwd):/app myapp
# Or check container user expectations

# Changes not reflected
# On Docker Desktop, try cache invalidation
docker-compose down
docker-compose up --force-recreate

# Slow performance on macOS
# Use :cached for reads, :delegated for writes
# Use volumes for node_modules, vendor, etc.

# Path issues on Windows
# Use forward slashes or escape backslashes
docker run -v "C:/Users/name/project:/app" myapp
docker run -v "C:\\Users\\name\\project:/app" myapp
```

## Interview Questions

### Q1: What is a bind mount?
**A:** A bind mount directly maps a file or directory from the host into a container. Changes are bidirectional and immediate. Unlike volumes, bind mounts depend on the host's directory structure and are not managed by Docker.

### Q2: When should you use bind mounts vs volumes?
**A:** Use bind mounts for development (live code reload, IDE access), configuration files, and when you need direct host file access. Use volumes for persistent data, databases, and production - they're managed by Docker and more portable.

### Q3: What happens if you bind mount a file that doesn't exist?
**A:** Docker creates a directory at that path instead of a file. This is a common gotcha. Always ensure the file exists before mounting, or use `--mount type=bind` which fails explicitly if the source doesn't exist.

### Q4: How do you solve file ownership issues with bind mounts?
**A:** Run the container as the host user with `--user $(id -u):$(id -g)`, or create a container user matching the host UID/GID in the Dockerfile. This ensures files created in the container have correct host ownership.

### Q5: Why are bind mounts slow on macOS/Windows?
**A:** Docker Desktop runs containers in a Linux VM. Bind mounts go through a file-sharing layer between host and VM, adding overhead. Use `:cached` or `:delegated` flags to improve performance, or use volumes for heavy I/O.

### Q6: How do you mount a read-only bind mount?
**A:** Add `:ro` suffix: `-v $(pwd)/config:/app/config:ro` or use `--mount type=bind,source=...,target=...,readonly`. The container can read but not modify the mounted files.

### Q7: What's the difference between -v and --mount syntax?
**A:** `-v` (short) creates directories if they don't exist and has less explicit options. `--mount` (long) is more explicit, fails if source doesn't exist (for bind mounts), and is recommended for clarity.

### Q8: How do you handle node_modules with bind mounts in development?
**A:** Don't bind mount node_modules - use a named volume: `-v $(pwd):/app -v node_modules:/app/node_modules`. This prevents host node_modules from overwriting container's and improves performance on Docker Desktop.

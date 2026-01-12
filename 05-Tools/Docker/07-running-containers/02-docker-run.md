---
tags: ['docker', 'containers', 'cli', 'devops', 'tools', 'roadmap']
---

# Docker Run Command

## Summary

The `docker run` command is the fundamental way to create and start containers from images. It provides extensive configuration options for networking, volumes, resource limits, security, and runtime behavior. Understanding `docker run` is essential for running containers in development, testing, and production environments.

## Detailed Explanation

### Basic Syntax

```bash
# MINIMAL RUN
docker run <image>

# WITH COMMAND
docker run <image> <command>

# EXAMPLES
docker run nginx
docker run alpine echo "Hello World"
docker run ubuntu:22.04 sleep 10
```

### Essential Flags

```bash
# BACKGROUND / FOREGROUND
-d, --detach          # Run in background (detached mode)
-f, --attach           # Attach to STDOUT/STDERR and forward signals

# INTERACTIVE
-i, --interactive      # Keep STDIN open even if not attached
-t, --tty             # Allocate a pseudo-TTY

# NAMING
--name <name>          # Container name (for reference)

# RESTART POLICY
--restart <policy>      # no, on-failure, always, unless-stopped

# AUTO REMOVE
--rm                  # Automatically remove container on exit

# WORKING DIRECTORY
-w, --workdir <dir>   # Working directory inside container

# USER
-u, --user <user>     # Run as specific user (UID:GID)

# ENVIRONMENT
-e, --env <VAR=value>  # Environment variable
--env-file <file>      # Read environment variables from file

# PORTS
-p, --publish <port>    # Publish container port to host
-P, --publish-all      # Publish all exposed ports

# VOLUMES
-v, --volume <mount>    # Bind mount or named volume
--mount <spec>         # Explicit mount specification

# RESOURCE LIMITS
-m, --memory <limit>   # Memory limit
--cpus <n.n>         # CPU limit
--pids-limit <n>       # Process ID limit

# NETWORKING
--network <net>         # Connect to network
--network-alias <alias> # Network alias

# HOSTNAME
--hostname <name>      # Container hostname

# HEALTHCHECK
--health-cmd <cmd>      # Health check command
--health-interval <sec>  # Time between health checks
--health-timeout <sec>   # Timeout for health check
--health-retries <n>      # Consecutive failures before unhealthy
--health-start-period <sec>  # Startup grace period

# READ-ONLY
--read-only            # Container root filesystem read-only

# PRIVILEGED
--privileged            # Extended privileges (DANGEROUS!)
```

### Common Usage Patterns

```bash
# WEB SERVER
docker run -d \
  --name web \
  -p 8080:80 \
  --restart unless-stopped \
  nginx:alpine

# INTERACTIVE SHELL
docker run -it \
  --name shell \
  --rm \
  ubuntu:22.04 \
  bash

# DEVELOPMENT WITH LIVE RELOAD
docker run -d \
  --name app \
  -p 3000:3000 \
  -v $(pwd):/app \
  -e NODE_ENV=development \
  node:20 \
  npm start

# DATABASE WITH PERSISTENCE
docker run -d \
  --name postgres \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  --restart unless-stopped \
  postgres:15

# ONE-OFF COMMAND
docker run --rm alpine cat /etc/os-release

# WITH CUSTOM COMMAND
docker run --rm python:3.11 python -c "print('Hello')"
```

### Port Mapping

```bash
# SINGLE PORT
docker run -p 8080:80 nginx
# Host 8080 → Container 80

# MULTIPLE PORTS
docker run -p 80:80 -p 443:443 nginx
# Host 80 → Container 80, Host 443 → Container 443

# BIND TO SPECIFIC INTERFACE
docker run -p 127.0.0.1:8080:80 nginx
# Only accessible on localhost

# BIND TO RANDOM PORT
docker run -p 80 nginx
# Docker assigns random host port (> 32768)
docker port <container>  # See assigned port

# UDP PORTS
docker run -p 53:53/udp coredns

# PUBLISH ALL EXPOSED PORTS
docker run -P nginx
# Maps all ports from Dockerfile EXPOSE instructions

# IPv6
docker run -p 8080:80/tcp6 nginx
```

### Volume Mounts

```bash
# NAMED VOLUME
docker run -v mydata:/app/data nginx
docker run --mount type=volume,source=mydata,target=/app/data nginx

# BIND MOUNT (HOST DIRECTORY)
docker run -v /host/path:/container/path nginx
docker run -v $(pwd):/app nginx
docker run -v "C:\Users\name\project:/app" nginx  # Windows

# READ-ONLY BIND MOUNT
docker run -v $(pwd)/config:/app/config:ro nginx

# MOUNTING SINGLE FILE
docker run -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# TMPFS (IN-MEMORY)
docker run --tmpfs /app/temp:size=100m nginx
docker run --mount type=tmpfs,destination=/app/temp,tmpfs-size=100m nginx

# MULTIPLE VOLUMES
docker run -v db_data:/var/lib/postgresql/data \
  -v logs_data:/var/log \
  postgres:15

# VOLUME OPTIONS
docker run -v mydata:/app/data:ro nginx
docker run -v /host/path:/container/path:cached nginx  # macOS optimization
docker run -v /host/path:/container/path:delegated nginx  # macOS optimization
```

### Environment Variables

```bash
# SINGLE VARIABLE
docker run -e APP_ENV=production nginx

# MULTIPLE VARIABLES
docker run \
  -e NODE_ENV=production \
  -e LOG_LEVEL=info \
  -e DATABASE_URL=postgres://db:5432/app \
  nginx

# FROM HOST ENVIRONMENT
export API_KEY=secret
docker run -e API_KEY nginx

# FROM FILE
docker run --env-file .env nginx
docker run --env-file production.env nginx

# QUOTING VALUES WITH SPACES
docker run -e MESSAGE="Hello World" nginx
docker run -e 'MESSAGE=Hello World' nginx

# EMPTY VALUE
docker run -e EMPTY="" nginx
docker run -e FLAG= nginx  # Sets to empty string
```

### Resource Limits

```bash
# MEMORY LIMIT
docker run -m 512m nginx
docker run --memory=512m nginx
docker run --memory=512m --memory-swap=1g nginx

# CPU LIMIT (DECIMAL CORES)
docker run --cpus=1.5 nginx
docker run --cpus=2.0 nginx

# CPU SHARES (RELATIVE WEIGHT)
docker run --cpu-shares=512 nginx
docker run --cpu-shares=1024 nginx  # Default is 1024
# Lower value = less CPU when contended

# CPU PINNING
docker run --cpuset-cpus=0,1 nginx  # Only use CPU 0 and 1

# MEMORY LIMITATION (SOFT + HARD)
docker run \
  --memory=512m \
  --memory-reservation=256m \
  nginx

# PID LIMIT (FORK BOMB PROTECTION)
docker run --pids-limit=100 nginx

# BLOCK I/O LIMITS
docker run \
  --device-read-bps=/dev/sda:10mb \
  --device-write-bps=/dev/sda:10mb \
  nginx

# COMBINED LIMITS
docker run -d \
  --name limited_app \
  --memory=512m \
  --cpus=1.0 \
  --pids-limit=200 \
  myapp:latest
```

### Networking Options

```bash
# USE DEFAULT BRIDGE NETWORK
docker run nginx  # Creates new bridge network

# USE SPECIFIC NETWORK
docker network create mynet
docker run --network mynet nginx

# HOST NETWORKING (NO ISOLATION)
docker run --network=host nginx
# Container shares host network stack

# NONE NETWORK (NO NETWORK)
docker run --network=none nginx
# Container has no network access

# ADD TO MULTIPLE NETWORKS
docker run --network bridge --network mynet nginx

# NETWORK ALIAS
docker run --network mynet --network-alias api nginx
# Accessible as "api" on mynet

# DNS CONFIGURATION
docker run --dns=8.8.8.8 --dns=8.8.4.4 nginx

# ADD HOSTS ENTRY
docker run --add-host=mydb:192.168.1.100 nginx

# SPECIFY MAC ADDRESS
docker run --mac-address=02:42:ac:11:00:02 nginx
```

### User and Security

```bash
# RUN AS SPECIFIC USER
docker run -u 1000:1000 nginx
docker run --user nobody nginx

# RUN AS NON-ROOT USER
docker run -u $(id -u):$(id -g) nginx

# DROP ALL CAPABILITIES
docker run --cap-drop=ALL nginx

# DROP ALL, ADD SPECIFIC
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# READ-ONLY ROOT FILESYSTEM
docker run --read-only nginx
docker run -v /tmp:/tmp --read-only nginx

# NO NEW PRIVILEGES
docker run --security-opt=no-new-privileges:true nginx

# SECCOMP PROFILE
docker run --security-opt seccomp=/path/to/profile.json nginx

# APPARMOR PROFILE
docker run --security-opt apparmor=docker-default nginx

# PRIVILEGED MODE (DANGEROUS)
docker run --privileged nginx  # Full access to host devices
# AVOID IN PRODUCTION!
```

### Health Checks

```bash
# SIMPLE HEALTH CHECK
docker run -d \
  --name web \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --health-timeout=5s \
  --health-retries=3 \
  nginx:alpine

# USING SHELL COMMAND
docker run -d \
  --name app \
  --health-cmd="pg_isready -U appuser" \
  --health-interval=10s \
  postgres:15

# CHECK HEALTH STATUS
docker inspect --format='{{.State.Health.Status}}' web
# Output: healthy, unhealthy, starting

# VIEW HEALTH LOGS
docker inspect --format='{{json .State.Health.Log}}' web | jq

# DISABLE HEALTH CHECK
docker run --health-cmd=NONE nginx
docker run --no-healthcheck nginx
```

### Working Directory

```bash
# SET WORKING DIRECTORY
docker run -w /app nginx
docker run --workdir /app nginx

# ABSOLUTE PATH
docker run -w /usr/local/app nginx

# RELATIVE TO IMAGE DEFAULT
docker run -w app nginx  # If image WORKDIR is /app

# OVERRIDE IMAGE WORKDIR
# If image sets WORKDIR /app
docker run -w /custom nginx  # Uses /custom instead
```

### Restart Policies

```bash
# NEVER RESTART (DEFAULT)
docker run --restart=no nginx

# RESTART ON FAILURE ONLY
docker run --restart=on-failure nginx

# RESTART ON FAILURE WITH LIMIT
docker run --restart=on-failure:5 nginx  # Max 5 attempts

# ALWAYS RESTART
docker run --restart=always nginx
# Restart on daemon start and container exit

# UNLESS STOPPED (RECOMMENDED)
docker run --restart=unless-stopped nginx
# Restart on daemon start and failure, but not manual stop
```

### Lifecycle Commands

```bash
# CONTAINER LIFECYCLE WITH docker run

# Create container
docker run -d --name myapp nginx

# View running containers
docker ps

# View all containers (including stopped)
docker ps -a

# Stop container
docker stop myapp

# Start stopped container
docker start myapp

# Restart container
docker restart myapp

# Remove container
docker rm myapp

# Remove running container (force)
docker rm -f myapp

# Run, then inspect
ID=$(docker run -d alpine sleep 1000)
docker inspect $ID

# RUN AND GET CONTAINER ID
CONTAINER_ID=$(docker run -d --name myapp nginx)
docker logs $CONTAINER_ID
```

### Special Use Cases

```bash
# DOCKER-IN-DOCKER
docker run -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker:latest \
  docker ps

# GPU ACCELERATION
docker run --gpus all nvidia/cuda:11.0-base-ubuntu20.04 nvidia-smi

# CUSTOM ENTRYPOINT
docker run --entrypoint=/bin/sh nginx -c "echo custom startup"

# OVERRIDE CMD
docker run --rm nginx echo "Hello World"
# Runs echo instead of default CMD

# PASS THROUGH
docker run -it --rm -p 53:53/udp \
  --net=host \
  coredns/coredns \
  -dns 127.0.0.1 \
  -conf /dev/stdin

# TEMPORARY CONTAINER FOR DEBUG
docker run --rm -it \
  -v $(pwd):/app \
  --entrypoint /bin/sh \
  nginx:alpine
```

## Interview Questions

### Q1: What is the difference between `-d` and `-it` flags?
**A:** `-d` (detached) runs container in background, returning control to shell immediately. `-it` (interactive + TTY) keeps STDIN open and allocates a pseudo-terminal, used for interactive shell sessions. You can combine `-d -it` but typically use one or the other.

### Q2: What does `--rm` flag do?
**A:** The `--rm` flag automatically removes the container when it exits. This is useful for one-off commands or temporary containers to prevent accumulation of stopped containers and save disk space.

### Q3: How do you map ports from container to host?
**A:** Use `-p host_port:container_port` or `-p host_port:container_port/protocol`. For example, `-p 8080:80` maps host port 8080 to container port 80. Use `-P` to publish all exposed ports to random host ports.

### Q4: What is the difference between named volumes and bind mounts?
**A:** Named volumes (`-v mydata:/path`) are Docker-managed storage in `/var/lib/docker/volumes/`, portable, and persist independent of container lifecycle. Bind mounts (`-v /host/path:/path`) map specific host directories into containers, useful for development but less portable.

### Q5: How do you set resource limits for containers?
**A:** Use `--memory=<limit>` for memory (e.g., `512m`, `1g`), `--cpus=<n>` for CPU (e.g., `1.5` for 1.5 cores), and `--pids-limit=<n>` to limit number of processes. Memory limits trigger OOM killer when exceeded; CPU limits throttle the process.

### Q6: What is `--restart=unless-stopped` policy?
**A:** The `unless-stopped` policy restarts containers on daemon startup and when they exit with non-zero exit codes, but NOT after you manually stop them with `docker stop`. This is the recommended policy for production services that should auto-recover but respect manual intervention.

### Q7: How do you run a container as a non-root user?
**A:** Use `-u <uid>:<gid>` or `-u username` flag. For example, `-u 1000:1000` runs as user ID 1000 with group ID 1000. Using `-u $(id -u):$(id -g)` runs as your current host user, which helps with file permissions on bind mounts.

### Q8: When would you use `--privileged` mode?
**A:** `--privileged` mode gives a container extended access to host devices and disables most security isolation. It's dangerous and should only be used for specific cases like running Docker-in-Docker, debugging system-level issues, or containers needing direct hardware access (e.g., GPUs). Never use it in production.

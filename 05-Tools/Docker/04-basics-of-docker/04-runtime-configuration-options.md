---
tags: ['docker', 'containers', 'devops', 'tools', 'roadmap']
---

# Runtime Configuration Options

## Summary

Docker provides extensive runtime configuration options through `docker run` flags that control container behavior for networking, resources, security, storage, and environment. Understanding these options is essential for running containers properly in development and production. Key categories include port mapping, volume mounts, environment variables, resource limits, restart policies, and security settings.

## Detailed Explanation

### Essential Run Options

```bash
# Complete example with common options
docker run -d \
  --name myapp \
  --hostname myapp-host \
  --restart unless-stopped \
  -p 8080:80 \
  -e APP_ENV=production \
  -e DATABASE_URL=postgres://db:5432/app \
  -v app_data:/app/data \
  -v ./config:/app/config:ro \
  --memory=512m \
  --cpus=1.0 \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --label app=myapp \
  --label env=prod \
  myapp:v1.0.0

# Breakdown of all options follows...
```

### Container Identity

```bash
# Name (for reference)
docker run --name mywebserver nginx
docker stop mywebserver
docker logs mywebserver

# Hostname (inside container)
docker run --hostname myapp nginx
docker exec myapp hostname  # myapp

# Network aliases
docker run --network mynet --network-alias api nginx
# Container reachable as "api" on mynet network

# Labels (metadata)
docker run \
  --label app=frontend \
  --label env=staging \
  --label version=1.2.3 \
  nginx

# Filter by labels
docker ps --filter "label=env=staging"
```

### Port Mapping

```bash
# Map single port
docker run -p 8080:80 nginx
# Host port 8080 -> Container port 80

# Map multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# Map to specific interface
docker run -p 127.0.0.1:8080:80 nginx  # localhost only
docker run -p 0.0.0.0:8080:80 nginx    # all interfaces (default)

# Random host port
docker run -p 80 nginx
docker port <container>  # See assigned port

# Expose all defined ports to random host ports
docker run -P nginx

# UDP ports
docker run -p 5000:5000/udp myapp

# View port mappings
docker port mycontainer
# 80/tcp -> 0.0.0.0:8080
```

### Environment Variables

```bash
# Single variable
docker run -e DATABASE_URL=postgres://localhost/db nginx

# Multiple variables
docker run \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  -e SECRET_KEY=abc123 \
  myapp

# From host environment
export API_KEY=secret
docker run -e API_KEY myapp  # Passes $API_KEY value

# From file
# .env file:
# DATABASE_URL=postgres://localhost/db
# API_KEY=secret123
docker run --env-file .env myapp

# View container environment
docker inspect myapp --format '{{json .Config.Env}}'
```

### Volume Mounts

```bash
# Named volume (Docker managed)
docker run -v mydata:/app/data nginx

# Bind mount (host path)
docker run -v /host/path:/container/path nginx
docker run -v $(pwd)/config:/app/config nginx

# Read-only mount
docker run -v ./config:/app/config:ro nginx

# tmpfs mount (memory)
docker run --tmpfs /app/temp nginx
docker run --mount type=tmpfs,destination=/app/temp,tmpfs-size=100m nginx

# Modern --mount syntax (more explicit)
docker run \
  --mount type=volume,source=mydata,target=/app/data \
  --mount type=bind,source=$(pwd)/config,target=/app/config,readonly \
  nginx

# Volume driver options
docker run \
  --mount type=volume,source=mydata,target=/data,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/path \
  nginx
```

### Resource Limits

```bash
# Memory
docker run --memory=512m nginx          # Hard limit
docker run --memory=512m --memory-swap=1g nginx  # With swap
docker run --memory-reservation=256m nginx       # Soft limit

# CPU
docker run --cpus=1.5 nginx             # 1.5 cores max
docker run --cpu-shares=512 nginx       # Relative weight
docker run --cpuset-cpus=0,1 nginx      # Pin to specific cores

# Combined limits
docker run \
  --memory=512m \
  --memory-swap=512m \
  --cpus=1.0 \
  --pids-limit=100 \
  nginx

# Block I/O
docker run \
  --device-read-bps=/dev/sda:10mb \
  --device-write-bps=/dev/sda:10mb \
  nginx

# View resource usage
docker stats
docker stats --no-stream myapp
```

### Restart Policies

```bash
# no (default) - Never restart
docker run --restart=no nginx

# on-failure - Restart on non-zero exit
docker run --restart=on-failure nginx
docker run --restart=on-failure:5 nginx  # Max 5 attempts

# always - Always restart
docker run --restart=always nginx
# Starts on daemon startup too

# unless-stopped - Like always, but not after manual stop
docker run --restart=unless-stopped nginx
# Won't start on daemon restart if manually stopped

# Production recommendation
docker run --restart=unless-stopped myapp
```

### Health Checks

```bash
# Health check options
docker run \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=60s \
  nginx

# Using wget (for Alpine)
docker run \
  --health-cmd="wget --quiet --tries=1 --spider http://localhost/health || exit 1" \
  nginx

# Check container health status
docker inspect --format='{{.State.Health.Status}}' myapp
# starting, healthy, unhealthy

# View health check history
docker inspect --format='{{json .State.Health}}' myapp | jq
```

### Networking

```bash
# Use specific network
docker run --network=mynetwork nginx

# Host networking (no isolation)
docker run --network=host nginx

# No networking
docker run --network=none nginx

# Connect to container network
docker run --network=container:other_container nginx

# DNS configuration
docker run \
  --dns=8.8.8.8 \
  --dns=8.8.4.4 \
  --dns-search=example.com \
  nginx

# Add /etc/hosts entry
docker run --add-host=myhost:192.168.1.100 nginx

# MAC address
docker run --mac-address=02:42:ac:11:00:02 nginx
```

### Security Options

```bash
# Run as specific user
docker run --user 1000:1000 nginx
docker run --user nobody nginx

# Read-only root filesystem
docker run --read-only nginx

# Drop all capabilities
docker run --cap-drop=ALL nginx

# Add specific capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Security options
docker run \
  --security-opt=no-new-privileges:true \
  --security-opt=seccomp=/path/to/profile.json \
  nginx

# AppArmor profile
docker run --security-opt apparmor=docker-default nginx

# SELinux labels
docker run --security-opt label=type:container_t nginx

# Privileged mode (AVOID in production)
docker run --privileged nginx  # Full host access!
```

### Execution Options

```bash
# Interactive mode
docker run -it nginx bash
docker run -i nginx  # Keep STDIN open
docker run -t nginx  # Allocate pseudo-TTY

# Detached (background)
docker run -d nginx

# Auto-remove on exit
docker run --rm nginx

# Working directory
docker run -w /app nginx

# Override entrypoint
docker run --entrypoint=/bin/sh nginx -c "echo hello"

# Override command
docker run nginx cat /etc/nginx/nginx.conf

# Init process (reap zombies)
docker run --init nginx

# Timeout
docker run --stop-timeout=30 nginx
```

### Logging Options

```bash
# Logging driver
docker run --log-driver=json-file nginx
docker run --log-driver=syslog nginx
docker run --log-driver=none nginx  # Disable logging

# Log options
docker run \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Log options for syslog
docker run \
  --log-driver=syslog \
  --log-opt syslog-address=tcp://192.168.0.42:123 \
  --log-opt syslog-facility=daemon \
  nginx
```

### Complete Production Example

```bash
# Production-ready container
docker run -d \
  --name api-server \
  --hostname api \
  --restart unless-stopped \
  --network production \
  --network-alias api \
  -p 8080:8080 \
  -e NODE_ENV=production \
  -e DATABASE_URL \
  -e REDIS_URL \
  --env-file .env.production \
  -v api_data:/app/data \
  -v ./certs:/app/certs:ro \
  --memory=1g \
  --memory-swap=1g \
  --cpus=2.0 \
  --pids-limit=200 \
  --user 1000:1000 \
  --read-only \
  --tmpfs /tmp \
  --security-opt=no-new-privileges:true \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --health-cmd="curl -f http://localhost:8080/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=60s \
  --log-driver=json-file \
  --log-opt max-size=100m \
  --log-opt max-file=5 \
  --label app=api \
  --label env=production \
  --label version=1.2.3 \
  mycompany/api:1.2.3
```

## Interview Questions

### Q1: What is the difference between `-p 8080:80` and `-P`?
**A:** `-p 8080:80` maps a specific host port (8080) to container port (80). `-P` publishes all exposed ports to random high ports on the host. Use `-p` for predictable ports, `-P` for temporary/testing scenarios.

### Q2: What restart policy should you use in production?
**A:** `--restart unless-stopped` is recommended. It restarts containers on failure and daemon restart, but respects manual stops. `always` also works but restarts containers even after manual stops.

### Q3: What is the difference between named volumes and bind mounts?
**A:** Named volumes (`-v mydata:/path`) are managed by Docker in its data directory, portable, and can use volume drivers. Bind mounts (`-v /host/path:/path`) map to specific host paths, useful for config files and development.

### Q4: How do you limit container memory and what happens when exceeded?
**A:** Use `--memory=512m` to set hard limit. When exceeded, the container's processes are killed by the OOM killer. Use `--memory-reservation` for soft limits that allow burst usage.

### Q5: What does `--read-only` do and why use it?
**A:** It makes the container's root filesystem read-only, preventing modifications. Use with `--tmpfs /tmp` for temp files. This improves security by preventing attackers from modifying the container filesystem.

### Q6: What is the purpose of `--cap-drop=ALL --cap-add=NET_BIND_SERVICE`?
**A:** This drops all Linux capabilities (reducing privilege) then adds back only the specific capability needed (binding to ports below 1024). This is the principle of least privilege - only grant what's needed.

### Q7: What is the difference between `-e VAR=value` and `--env-file`?
**A:** `-e VAR=value` sets individual variables inline (visible in process lists and history). `--env-file .env` reads variables from a file, better for multiple variables and keeping secrets out of command history.

### Q8: When would you use `--init` flag?
**A:** When your container runs a process that may spawn children but doesn't handle zombie reaping. `--init` adds a minimal init process (tini) as PID 1 that properly reaps zombie processes.

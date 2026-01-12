#Linux
---
tags: ['linux', 'roadmap', 'tools', 'docker', 'containers']
---

## Summary
Docker is an open-source platform that automates the deployment, scaling, and management of applications within lightweight, portable containers. Unlike Virtual Machines, Docker containers share the host's OS kernel, utilizing Linux kernel features like **Namespaces** for isolation and **Cgroups** for resource allocation. This note covers Docker fundamentals (Images, Containers, Dockerfiles), data persistence, networking, and multi-container orchestration with Docker Compose.

## Detailed Explanation

### 1. Images vs. Containers
*   **Image**: A read-only template containing the application code, libraries, dependencies, and environment variables. It is composed of multiple layers.
*   **Container**: A runnable instance of an image. It adds a thin **writable layer** on top of the read-only image layers.

### 2. The Dockerfile: Building Images
A Dockerfile is a text document containing all the commands a user could call on the command line to assemble an image.

```dockerfile
# Use a lightweight base image
FROM alpine:3.18

# Set the working directory
WORKDIR /app

# Copy application files
COPY . .

# Install dependencies (example)
RUN apk add --no-cache bash

# Set environment variables
ENV APP_PORT=8080

# Expose the port
EXPOSE 8080

# The command to run when the container starts
CMD ["bash", "start.sh"]
```

**Bash Example: Building and Tagging**
```bash
# Build an image from the current directory and tag it
docker build -t my-app:v1.0 .

# List local images
docker images
```

### 3. Container Lifecycle: Run, Stop, Push
```bash
# RUN: Start a container in detached mode with port mapping
docker run -d -p 8080:8080 --name my-running-app my-app:v1.0

# STOP: Gracefully stop a running container
docker stop my-running-app

# RM: Remove a stopped container
docker rm my-running-app

# PUSH: Upload an image to a registry (requires login)
docker tag my-app:v1.0 my-username/my-app:v1.0
docker push my-username/my-app:v1.0
```

### 4. Data Persistence: Volumes
Containers are ephemeral; data is lost when a container is removed unless persisted.

*   **Volumes**: Managed by Docker (stored in `/var/lib/docker/volumes/`). Best for production.
*   **Bind Mounts**: Maps a specific path on the host to a path in the container.

```bash
# Create a volume
docker volume create my-data

# Run container with a volume
docker run -d --name db -v my-data:/var/lib/mysql mysql:latest

# Run container with a bind mount (ideal for development)
docker run -d -v $(pwd)/src:/app/src my-app:v1.0
```

### 5. Networking
Docker creates a default bridge network (`docker0`), but custom networks are preferred for service discovery.

```bash
# Create a custom bridge network
docker network create my-network

# Connect containers to the network (allows communication by container name)
docker run -d --name web --network my-network my-web-app
docker run -d --name api --network my-network my-api-app
```

### 6. Docker Compose: Multi-Container Orchestration
Docker Compose allows you to define and run multi-container applications using a YAML file.

**docker-compose.yml**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "80:8080"
    networks:
      - frontend
    depends_on:
      - db
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - frontend
    environment:
      POSTGRES_PASSWORD: example_password

volumes:
  db-data:

networks:
  frontend:
```

**Bash Example: Compose Commands**
```bash
# Start all services defined in the YAML file
docker compose up -d

# Check status
docker compose ps

# Stop and remove all containers/networks defined
docker compose down
```

### 7. Linux Internals (Advanced)
*   **Namespaces**: Provide isolation (PID, NET, MNT, UTS, IPC).
*   **Cgroups**: Manage resource limits (CPU, Memory).
*   **Overlay2**: The default storage driver that handles the layered filesystem.

## Interview Questions

**Q: What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?**
**A:** `ENTRYPOINT` defines the executable that runs when the container starts and is not easily overridden. `CMD` provides default arguments for the `ENTRYPOINT` or a default command. If both are used, `CMD` is appended to `ENTRYPOINT`.

**Q: How do you reduce the size of a Docker image?**
**A:** Use smaller base images (like Alpine or Distroless), use **multi-stage builds** to exclude build tools from the final image, minimize the number of `RUN` layers by combining commands, and use `.dockerignore` to exclude unnecessary files.

**Q: What is the difference between a Volume and a Bind Mount?**
**A:** Volumes are managed by Docker and are more portable and secure, as they are isolated from the host's direct filesystem structure. Bind mounts depend on the host's directory structure and are often used for mapping source code during development.

**Q: How do containers communicate with each other in Docker?**
**A:** Containers on the same user-defined network can communicate with each other using their container names as hostnames via Docker's internal DNS.

**Q: What happens to the data inside a container when it is stopped? What about when it is deleted?**
**A:** When stopped, the data remains in the container's writable layer. When deleted, all data in the writable layer is lost forever, which is why persistent data must be stored in Volumes or Bind Mounts.

---
tags: ['docker', 'containers', 'linux', 'devops', 'tools', 'roadmap']
---

# Docker Engine

## Summary

Docker Engine is the core runtime that runs and manages containers on Linux systems. Unlike Docker Desktop (which includes a GUI and VM), Docker Engine runs natively on Linux and consists of the Docker daemon (dockerd), Docker CLI, and container runtime (containerd + runc). It's free, open-source, and the standard for production container workloads on Linux servers.

## Detailed Explanation

### Docker Engine Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Docker CLI                            │
│                    (docker commands)                         │
└─────────────────────────────────────────────────────────────┘
                            │
                      REST API
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     Docker Daemon                            │
│                       (dockerd)                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Image Management  │  Network  │  Volume  │  Build     ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      containerd                              │
│            (container lifecycle management)                  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                         runc                                 │
│           (OCI runtime - creates containers)                 │
└─────────────────────────────────────────────────────────────┘
```

### Installation on Ubuntu/Debian

```bash
# Remove old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
sudo docker run hello-world
```

### Installation on RHEL/CentOS/Fedora

```bash
# Remove old versions
sudo yum remove docker docker-client docker-client-latest \
  docker-common docker-latest docker-latest-logrotate \
  docker-logrotate docker-engine

# Install prerequisites (Fedora)
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo \
  https://download.docker.com/linux/fedora/docker-ce.repo

# Install (Fedora)
sudo dnf install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# For CentOS/RHEL, use yum and appropriate repo URL

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify
sudo docker run hello-world
```

### Post-Installation Setup

```bash
# Run Docker without sudo (recommended for development)
sudo groupadd docker
sudo usermod -aG docker $USER

# Apply group membership (logout/login or run:)
newgrp docker

# Verify non-root access
docker run hello-world

# Enable Docker to start on boot
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

# Configure Docker to use systemd cgroup driver (for Kubernetes)
sudo mkdir -p /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Configuration (daemon.json)

```json
// /etc/docker/daemon.json
{
  // Storage configuration
  "storage-driver": "overlay2",
  "data-root": "/var/lib/docker",
  
  // Logging configuration
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  
  // Network configuration
  "bip": "172.17.0.1/16",
  "default-address-pools": [
    {"base": "172.18.0.0/16", "size": 24}
  ],
  "dns": ["8.8.8.8", "8.8.4.4"],
  
  // Security
  "userns-remap": "default",
  "no-new-privileges": true,
  
  // Performance
  "live-restore": true,
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 5,
  
  // Registry configuration
  "insecure-registries": ["registry.internal:5000"],
  "registry-mirrors": ["https://mirror.gcr.io"],
  
  // Cgroup driver (for Kubernetes)
  "exec-opts": ["native.cgroupdriver=systemd"],
  
  // Enable experimental features
  "experimental": true,
  
  // BuildKit
  "features": {
    "buildkit": true
  }
}
```

```bash
# Validate and apply configuration
sudo dockerd --validate
sudo systemctl restart docker
docker info
```

### Managing Docker Service

```bash
# Service management
sudo systemctl start docker
sudo systemctl stop docker
sudo systemctl restart docker
sudo systemctl status docker

# View Docker daemon logs
sudo journalctl -u docker.service
sudo journalctl -u docker.service -f  # Follow

# Check Docker daemon configuration
docker info

# Docker system information
docker version
docker system info
docker system df  # Disk usage

# Cleanup
docker system prune  # Remove unused data
docker system prune -a --volumes  # More aggressive cleanup
```

### Rootless Mode

```bash
# Run Docker daemon as non-root user
# Provides better security isolation

# Prerequisites
sudo apt-get install uidmap dbus-user-session

# Install rootless Docker
dockerd-rootless-setuptool.sh install

# Set environment variables (add to ~/.bashrc)
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

# Start rootless daemon
systemctl --user start docker
systemctl --user enable docker

# Verify
docker run hello-world

# Rootless limitations:
# - Cannot use --privileged
# - Limited network options
# - Some storage drivers not available
# - Cannot bind to ports < 1024 (without capabilities)
```

### Remote Access Configuration

```bash
# Enable remote Docker API access (CAUTION: security risk)

# Option 1: TCP without TLS (INSECURE - for testing only)
# /etc/docker/daemon.json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"]
}

# Option 2: TCP with TLS (recommended for remote access)
# Generate certificates
mkdir -p ~/.docker/certs

# Create CA
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

# Create server certificate
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=docker-server" -sha256 -new -key server-key.pem -out server.csr
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem

# Configure daemon with TLS
# /etc/docker/daemon.json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "tls": true,
  "tlscacert": "/etc/docker/certs/ca.pem",
  "tlscert": "/etc/docker/certs/server-cert.pem",
  "tlskey": "/etc/docker/certs/server-key.pem",
  "tlsverify": true
}

# Connect remotely
export DOCKER_HOST=tcp://docker-server:2376
export DOCKER_TLS_VERIFY=1
export DOCKER_CERT_PATH=~/.docker/certs
docker info
```

### Docker Context

```bash
# Manage multiple Docker endpoints
docker context ls
# NAME        DESCRIPTION                               DOCKER ENDPOINT
# default *   Current DOCKER_HOST based configuration   unix:///var/run/docker.sock

# Create context for remote Docker
docker context create remote-server \
  --docker "host=tcp://192.168.1.100:2376,ca=~/.docker/ca.pem,cert=~/.docker/cert.pem,key=~/.docker/key.pem"

# Switch context
docker context use remote-server

# Use specific context for single command
docker --context remote-server ps

# Remove context
docker context rm remote-server
```

### Troubleshooting

```bash
# Docker daemon won't start
sudo journalctl -u docker.service --no-pager
sudo dockerd --debug  # Run in foreground with debug

# Check permissions
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker 0 Jan 1 00:00 /var/run/docker.sock

# User not in docker group
id $USER  # Check groups
sudo usermod -aG docker $USER
newgrp docker

# Storage issues
docker system df
df -h /var/lib/docker
docker system prune -a --volumes

# Network issues
docker network ls
docker network prune
sudo iptables -L -n  # Check firewall rules

# Container issues
docker logs <container>
docker inspect <container>
docker events  # Real-time events

# Reset Docker (last resort)
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
sudo systemctl start docker
```

## Interview Questions

### Q1: What is the difference between Docker Desktop and Docker Engine?
**A:** Docker Engine is the core runtime for Linux (daemon + CLI + containerd). Docker Desktop is a commercial application for macOS/Windows that includes Docker Engine in a VM plus GUI, Kubernetes, and developer tools. Docker Engine is free; Docker Desktop has licensing requirements for large organizations.

### Q2: What components make up Docker Engine?
**A:** Docker Engine consists of: dockerd (daemon handling API requests), Docker CLI (user interface), containerd (container lifecycle management), and runc (OCI runtime that creates containers). The CLI communicates with dockerd via REST API.

### Q3: How do you run Docker commands without sudo?
**A:** Add your user to the docker group: `sudo usermod -aG docker $USER`, then log out and back in (or run `newgrp docker`). This gives the user access to the Docker socket at `/var/run/docker.sock`.

### Q4: What is rootless Docker mode?
**A:** Rootless mode runs the Docker daemon as a non-root user, providing better security by not requiring root privileges. It uses user namespaces for isolation but has some limitations (no privileged containers, limited networking options).

### Q5: Where is Docker Engine configured?
**A:** Configuration is in `/etc/docker/daemon.json`. This controls storage driver, logging, network settings, registry mirrors, TLS, and other daemon options. Restart Docker after changes: `sudo systemctl restart docker`.

### Q6: How do you enable remote Docker API access securely?
**A:** Configure TLS certificates and set `tls`, `tlsverify`, `tlscert`, `tlskey`, and `tlscacert` in daemon.json. Enable TCP listener on port 2376 (TLS). Never expose Docker API without TLS in production - it allows full system access.

### Q7: What is `live-restore` in Docker configuration?
**A:** `live-restore: true` allows containers to keep running during Docker daemon restarts or upgrades. Without it, stopping dockerd stops all containers. Important for high-availability deployments.

### Q8: How do you connect to multiple Docker hosts?
**A:** Use Docker contexts: `docker context create <name> --docker "host=..."`. Switch contexts with `docker context use <name>`. This allows managing multiple Docker environments (local, remote, staging, production) from one CLI.

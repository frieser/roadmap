---
tags: ['docker', 'containers', 'networking', 'cli', 'devops', 'tools', 'roadmap']
---

# Docker Networks CLI

## Summary

Docker networking enables containers to communicate with each other and the outside world. Docker provides several network drivers including bridge (default), host (no isolation), overlay (Swarm), and macvlan/ipvlan for advanced configurations. The network CLI commands allow creating, inspecting, connecting, and managing container networks.

## Detailed Explanation

### Network Types

```
┌─────────────────────────────────────────────────────────────┐
│                    HOST NETWORK STACK                    │
│  ┌──────────────────────────────────────────────────────┐│
│  │  DOCKER BRIDGE NETWORK                     ││
│  │  Containers: bridge networks                   ││
│  │ 172.17.0.0/16 - Default bridge network      ││
│  │ 172.18.0.0/16 - Additional bridge networks   ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  DOCKER HOST NETWORK (container uses host)   ││
│  │  Container shares host's network stack       ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  DOCKER OVERLAY NETWORK (multi-host)        ││
│  │  Encrypted inter-container communication         ││
│  │  Used by Docker Swarm                         ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  NULL NETWORK (isolation)                    ││
│  │  No network access                            ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  MACVLAN / IPVLAN (advanced)             ││
│  │  Container gets direct Layer 2 MAC address  ││
│  │  Works with existing switches/routers         ││
│  └──────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### Creating Networks

```bash
# CREATE BRIDGE NETWORK (DEFAULT TYPE)
docker network create mynet

# CREATE NETWORK WITH SUBNET
docker network create --subnet=172.20.0.0/24 mynet

# CREATE NETWORK WITH DRIVER SPECIFICATION
docker network create --driver bridge mynet
docker network create --driver overlay mynet  # For Swarm

# CREATE NETWORK WITH IP RANGE
docker network create --subnet=172.20.0.0/16 --ip-range=172.20.10.0/24 mynet

# CREATE NETWORK WITH IPv6 SUBNET
docker network create --subnet=2001:db8::/64 mynet

# CREATE NETWORK WITH LABELS
docker network create --label env=staging --label project=myapp mynet

# CREATE NETWORK WITH OPTIONS
docker network create --driver bridge \
  --opt com.docker.network.bridge.enable_icc=true \
  --opt com.docker.network.bridge.name=br-mynet \
  mynet
```

### Listing Networks

```bash
# LIST ALL NETWORKS
docker network ls
# NETWORK ID    NAME            DRIVER    SCOPE
# abc123       bridge          bridge    local
# def456       host            host      local

# LIST WITH FORMATTING
docker network ls --format "table {{.Name}}\t{{.Driver}}\t{{.Scope}}"

# LIST ONLY NETWORK NAMES
docker network ls --format "{{.Name}}"

# LIST BY DRIVER
docker network ls --filter driver=bridge
docker network ls --filter driver=overlay

# LIST CUSTOM NETWORKS ONLY
docker network ls --filter type=custom
```

### Connecting Containers to Networks

```bash
# CONNECT TO SPECIFIC NETWORK
docker network connect mynet mycontainer

# CONNECT MULTIPLE CONTAINERS
docker network connect mynet web1 web2 web3

# CONNECT WITH ALIASES
docker network connect --alias api mynet myapp
# Container can be reached as "api" on mynet

# CONNECT TO HOST NETWORK (NO ISOLATION)
docker network connect host mycontainer
# Container shares host's network stack

# CONNECT WITH CONTAINER NAME OVERWRITE
docker network connect --alias othername mynet myexistingcontainer
# Overwrites existing alias for container
```

### Disconnecting from Networks

```bash
# DISCONNECT FROM SPECIFIC NETWORK
docker network disconnect mynet mycontainer

# DISCONNECT FROM ALL NETWORKS
docker network disconnect mycontainer
# Disconnects from all networks

# DISCONNECT MULTIPLE CONTAINERS
docker network disconnect mynet container1 container2 container3

# FORCE DISCONNECT
docker network disconnect -f mynet mycontainer
# Doesn't check if network exists
```

### Inspecting Networks

```bash
# INSPECT SPECIFIC NETWORK
docker network inspect mynet
# Shows full JSON configuration

# INSPECT NETWORK SUBNET
docker network inspect --format '{{range .IPAM.Config}}{{.Subnet}}{{"\n"}}{{end}}' mynet
# 172.20.0.0/24

# INSPECT GATEWAY
docker network inspect --format '{{range .IPAM.Config}}{{.Gateway}}{{"\n"}}{{end}}' mynet
# 172.20.0.1

# INSPECT CONNECTED CONTAINERS
docker network inspect --format '{{range .Containers}}{{.Name}}{{"\n"}}{{end}}' mynet

# INSPECT NETWORK LABELS
docker network inspect --format '{{.Labels}}' mynet
# map[env:production project:myapp]
```

### Removing Networks

```bash
# REMOVE SPECIFIC NETWORK
docker network rm mynet

# REMOVE MULTIPLE NETWORKS
docker network rm mynet1 mynet2 dbnet

# FORCE REMOVE (WITH CONTAINERS)
docker network rm -f mynet
# Can fail if containers are still connected

# REMOVE ALL UNUSED NETWORKS
docker network prune
# Removes all networks not used by any container

# REMOVE UNUSED WITH CONFIRMATION
docker network prune

# REMOVE NETWORKS WITH FILTERS
docker network prune --filter "until=24h"
# Networks not used in last 24 hours

# REMOVE NETWORKS BY LABEL
docker network prune --filter "label=env=staging"
# Remove all networks with specific label
```

### Bridge Network Details

```bash
# DEFAULT BRIDGE NETWORK (docker0)
docker network inspect bridge
# Subnet: 172.17.0.0/16
# Gateway: 172.17.0.1
# All containers connect to this unless specified otherwise

# VIEW BRIDGE IP ASSIGNMENTS
docker network inspect bridge --format '{{range .Containers}}{{.IPv4Address}}: {{.Name}}{{"\n"}}{{end}}'

# CONTAINER-TO-CONTAINER COMMUNICATION
docker run -d --name web nginx
docker run -d --link web:api api
# api container can reach "web" as hostname
# Uses /etc/hosts entry
```

### Host Network Mode

```bash
# RUN WITH HOST NETWORKING
docker run --network=host nginx
# Container uses host's network stack directly
# Same IP as host, can bind to <1024

# BENEFITS:
# - No NAT overhead, better performance
# - Direct access to host services
# - Easier debugging with host tools

# RISKS:
# - No network isolation
# - Container can access any host port
# - Security implications
# - Should not be used for untrusted containers

# USE CASES:
# - Monitoring agents
# - System-level services
# - Development with host resources
# - Debugging network issues
```

### Advanced Network Modes

```bash
# MACVLAN (802.1Q VLAN TRUNKING)
docker network create -d macvlan \
  --driver macvlan \
  --subnet=192.168.1.0/24 \
  -o parent=eth0 \
  mymacvlan

# Container usage:
docker run -d --network mymacvlan --ip=192.168.1.100 myapp
# Container gets direct Layer 2 access, can be on VLAN

# IPVLAN (L3 ROUTING)
docker network create -d ipvlan \
  --driver ipvlan \
  --subnet=192.168.2.0/24 \
  -o parent=eth0 \
  myipvlan

# Container usage:
docker run -d --network myipvlan myapp
# Container appears directly on Layer 3 network
```

### Overlay Network (Swarm)

```bash
# OVERLAY NETWORK (ENCRYPTED MULTI-HOST)
docker network create --driver overlay --attachable mynet

# SWARM-LEVEL NETWORK
docker network create --driver overlay --opt encrypted mynet

# ATTACH TO SPECIFIC SERVICE
docker service create --network mynet --replicas 3 myservice
# All service containers can communicate securely

# VIEW OVERLAY ENCRYPTION KEYS
docker network inspect mynet
# Shows encryption configuration
```

### Network Troubleshooting

```bash
# CHECK NETWORK CONNECTIVITY
docker network inspect mynet | grep -i driver

# CONTAINER NETWORK DEBUGGING
docker exec mynet ip addr
docker exec mynet ip route

# LIST ALL BRIDGE INTERFACES
ip link show
ip -br show

# CHECK DNS RESOLUTION
docker exec myapp nslookup myservice.internal
docker exec myapp cat /etc/resolv.conf

# NETWORK CONFLICTS
# If two containers use same host port
docker run -p 80:80 web1
docker run -p 80:80 web2  # Second container will fail to start

# CHECK PORT BINDINGS
docker port myapp
# Shows which ports are mapped to host

# VIEW CONTAINER IP ADDRESSES
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}: {{.Name}}{{"\n"}}{{end}}' myapp
```

### DNS Configuration

```bash
# CONTAINER DNS (OVERRIDE HOST DNS)
docker run --dns=8.8.8.8 --dns=8.8.4.4 nginx
# Container uses custom DNS servers

# SPECIFIC SEARCH DOMAIN
docker run --dns-search=mydomain.local nginx

# DNS OPTIONS IN DAEMON
# /etc/docker/daemon.json:
{
  "dns": ["8.8.8.8", "8.8.4.4"],
  "dns-search": ["mydomain.local"],
  "dns-opts": ["timeout:2", "attempts:1"]
}
```

## Interview Questions

### Q1: What is the default Docker network driver?
**A:** The default driver is `bridge`, which creates a private internal network for each container. Containers get their own IP address on the `172.17.0.0/16` subnet and can communicate through NAT unless configured otherwise.

### Q2: What is host networking mode and when would you use it?
**A:** Host networking (`--network=host`) removes network isolation entirely - the container shares the host's network stack. Use for monitoring agents, system-level services, or when maximum performance is needed. Don't use for untrusted containers.

### Q3: How do containers communicate on the same bridge network?
**A:** Containers on the same Docker bridge network can communicate with each other by container name. Docker provides built-in DNS resolution where container names resolve to their IP addresses. Use `--link` for legacy alias-based communication.

### Q4: What is the difference between `docker network disconnect` and `docker network rm`?
**A:** `docker network disconnect` removes a container from a network but keeps the network. `docker network rm` deletes the entire network. Use `disconnect` when you want to remove a container from a network, use `rm` when the network is no longer needed.

### Q5: What are MACVLAN and IPVLAN networks used for?
**A:** MACVLAN and IPVLAN are advanced network drivers that give containers direct Layer 2 or Layer 3 network access. MACVLAN provides 802.1Q VLAN trunking, while IPVLAN supports Layer 3 routing. Used for network-intensive workloads requiring direct network access.

### Q6: How do you create a network with a custom subnet?
**A:** Use `docker network create --subnet=172.20.0.0/24 mynet`. You can also specify `--gateway`, `--ip-range`, and `--aux-address`. Ensure subnets don't overlap with existing networks.

### Q7: What is the purpose of overlay networks?
**A:** Overlay networks are used by Docker Swarm for multi-host deployments. They provide encrypted communication between containers across different Docker hosts. Each container on an overlay network can communicate securely with containers on any host in the Swarm.

### Q8: How do you configure DNS for containers?
**A:** You can override DNS on a per-container basis with `--dns` flag, or globally in the Docker daemon configuration (`/etc/docker/daemon.json` under `"dns"`). Custom DNS takes precedence over host DNS.

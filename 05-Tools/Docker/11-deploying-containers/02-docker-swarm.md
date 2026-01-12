---
---

## Summary
Docker Swarm is Docker's native clustering and orchestration engine. It allows you to turn a pool of Docker hosts into a single virtual Docker host. While Kubernetes has largely won the "Orchestration War," Swarm remains popular for simpler use cases due to its ease of setup and integration with the standard Docker CLI.

## Detailed Explanation

### Architecture
*   **Manager Nodes**: Handle cluster state (Raft consensus), scheduling, and API.
*   **Worker Nodes**: Execute the containers (Tasks).

### Key Concepts
*   **Service**: The definition of a workload (e.g., "Run 3 replicas of Nginx").
*   **Stack**: A group of services defined in a `docker-compose.yml` file, deployed together.
*   **Overlay Network**: A virtual network spanning all nodes, allowing containers on different hosts to communicate securely via private IPs.

### vs Kubernetes
*   **Pros**: Built-in (no install), simple (`docker swarm init`), standard Docker API.
*   **Cons**: Fewer features, smaller ecosystem, less configurable scaling logic.

## Go-Specific Context/Examples

Deploying a Go app stack.

### Example: docker-compose.yml for Swarm
```yaml
version: "3.8"
services:
  web:
    image: myuser/go-app:v1
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
    ports:
      - "80:8080"
```
Deploy: `docker stack deploy -c docker-compose.yml myapp`.

## Interview Questions

**Q: What happens if a Manager node fails?**
**A:** If the cluster has enough managers to maintain Quorum (Raft), it continues working. If quorum is lost (e.g., 1 out of 1 managers down, or 2 out of 3), the cluster enters read-only mode (containers keep running, but you can't schedule new ones).

**Q: What is the "Routing Mesh"?**
**A:** It allows you to access a service port on *any* node in the swarm, even if the container is not running on that specific node. Swarm automatically routes the request to an active container on another node.

**Q: Can you run Swarm and Kubernetes on the same nodes?**
**A:** Technically yes (Docker Enterprise did this), but it's not recommended due to resource contention and networking complexity.

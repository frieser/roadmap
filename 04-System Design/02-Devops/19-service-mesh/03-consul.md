---
---

# Consul

Consul, by HashiCorp, is a multi-cloud service networking platform. It started as a **Service Discovery** tool but evolved into a full **Service Mesh** (Consul Connect).

## Summary

Consul is unique because it supports both **Kubernetes** and **Legacy (VM/Bare Metal)** environments equally well. It uses a central registry (Key-Value store) to track services. The Service Mesh feature (Connect) uses Envoy sidecars to encrypt and authorize traffic between services using "Intentions".

## Detailed Explanation

### 1. Service Discovery
Services register themselves with Consul (via API or config). Clients query Consul (DNS or API) to find the IP address of healthy service instances.

### 2. Consul Connect (Mesh)
*   **Identity**: Based on SPIFFE X.509 certificates.
*   **Intentions**: Policies that define which services can talk to each other (e.g., `Web allow DB`, `Web deny Legacy`). This is "Infrastructure as Code" for firewall rules.

---

## Go Implementation Example

Using the official `hashicorp/consul/api` to register a service and perform service discovery. This is often used in non-mesh scenarios or to bootstrap the mesh.

```go
package main

import (
	"fmt"
	"log"

	"github.com/hashicorp/consul/api"
)

func main() {
	// 1. Connect to local Consul Agent
	config := api.DefaultConfig()
	client, err := api.NewClient(config)
	if err != nil {
		log.Fatal(err)
	}

	// 2. Register a Service
	registration := &api.AgentServiceRegistration{
		ID:      "my-go-service-1",
		Name:    "go-service",
		Port:    8080,
		Address: "192.168.1.5",
		Check: &api.AgentServiceCheck{
			HTTP:     "http://192.168.1.5:8080/health",
			Interval: "10s",
		},
	}

	err = client.Agent().ServiceRegister(registration)
	if err != nil {
		log.Fatal("Registration failed: ", err)
	}
	fmt.Println("Service Registered!")

	// 3. Service Discovery (Find 'db-service')
	services, _, err := client.Catalog().Service("db-service", "", nil)
	if err != nil {
		log.Fatal(err)
	}

	for _, s := range services {
		fmt.Printf("Found DB at: %s:%d\n", s.ServiceAddress, s.ServicePort)
	}
}
```

## Interview Questions

**Q: How does Consul differ from Etcd?**
**A:** Both are distributed Key-Value stores using Raft consensus.
*   **Etcd**: Designed primarily as the backing store for Kubernetes. Very simple API.
*   **Consul**: Designed as a Service Discovery system first. It includes high-level features like Health Checking, DNS interface, and Multi-Datacenter federation out of the box.

**Q: What is a "Gossip Protocol" (SERF) in Consul?**
**A:** Consul uses a Gossip Protocol (SWIM) to manage cluster membership and detect failures. Agents talk to random other agents to spread information (like "Node A is down"). This is much more scalable than having a central server heartbeat every single node, allowing Consul clusters to scale to thousands of nodes.

**Q: What are "Intentions" in Consul Connect?**
**A:** Intentions are authorization rules for the service mesh. They define access control at the *service* level, not the IP level. Example: `Allow frontend to connect to backend`. This decouples security from network topology (IP addresses), making it easier to secure dynamic cloud environments.

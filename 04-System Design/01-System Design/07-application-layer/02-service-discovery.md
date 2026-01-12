---
---

# Service Discovery

## Summary
Service discovery is the process of automatically detecting devices and services on a computer network. In a microservices architecture, it allows services to find and communicate with each other dynamically, ensuring that requests are always routed to healthy and available instances.

## Detailed Development

### **The Problem**
In modern cloud and containerized environments, service instances are ephemeral. They are frequently scaled up or down, moved between hosts, or restarted due to failures. This results in **dynamic IP addresses and ports**. Relying on static configuration files is impossible, as they would require constant manual updates, leading to downtime and configuration drift.

### **Client-Side Discovery**
- **Mechanism**: The service client is responsible for determining the network locations of available service instances and load balancing requests across them.
- **Workflow**:
    1. The client queries the **Service Registry** (a database of service locations).
    2. The registry returns a list of healthy instances for the requested service.
    3. The client selects an instance (using a load balancing algorithm) and makes the network call.
- **Pros**: Faster communication (one fewer network hop), and the client can implement specialized load balancing logic.
- **Cons**: The client must include discovery logic for every language/platform used; tightly coupled to the service registry.
- **Example**: Netflix Eureka with Ribbon.

### **Server-Side Discovery**
- **Mechanism**: The client makes a request to a service via a **Load Balancer** (or API Gateway/Router). The load balancer queries the service registry and routes the request to an available instance.
- **Workflow**:
    1. The client sends a request to a fixed endpoint (the Load Balancer).
    2. The Load Balancer queries the **Service Registry**.
    3. The Load Balancer routes the request to a healthy instance.
- **Pros**: Keeps the client code simple and language-agnostic; centralized management of routing and security.
- **Cons**: Introduces an additional network hop; the load balancer must be highly available and managed.
- **Example**: AWS Elastic Load Balancer (ELB), Kubernetes Services (Kube-proxy).

## Tools

### **Consul**
Consul (by HashiCorp) is a distributed, highly available system for service discovery and configuration.
- **Key Features**: HTTP and DNS-based service discovery, robust health checking, a distributed Key/Value store, and multi-datacenter support.

### **Etcd**
Etcd is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system.
- **Context**: It is famously used as the "source of truth" for Kubernetes, storing all cluster state and configuration.

### **ZooKeeper**
Apache ZooKeeper is a centralized service for maintaining configuration information, naming, and providing distributed synchronization.
- **Context**: Widely used in traditional distributed systems (e.g., Hadoop, Kafka) for coordination and leader election.

## Go-Specific Applications

In Go, service discovery is typically handled via specialized libraries or frameworks:
- **Go-kit**: Includes a dedicated `sd` (service discovery) package that provides adapters for Consul, Etcd, ZooKeeper, and Eureka, allowing you to wrap endpoints with discovery logic.
- **Go-micro**: Features a pluggable `Registry` interface. While it defaults to mDNS (Multicast DNS) for zero-config local discovery, it can be easily configured to use Consul or Etcd for production environments.

## Go Code Example (Consul Registration)

```go
package main

import (
	"log"
	"github.com/hashicorp/consul/api"
)

func main() {
	// 1. Initialize Consul Client
	config := api.DefaultConfig()
	client, err := api.NewClient(config)
	if err != nil {
		log.Fatal(err)
	}

	// 2. Define Service Registration
	registration := &api.AgentServiceRegistration{
		ID:      "order-service-1",
		Name:    "order-service",
		Port:    8080,
		Address: "192.168.1.10",
		Check: &api.AgentServiceCheck{
			HTTP:     "http://192.168.1.10:8080/health",
			Interval: "10s",
			Timeout:  "5s",
		},
	}

	// 3. Register the Service
	err = client.Agent().ServiceRegister(registration)
	if err != nil {
		log.Fatalf("Failed to register service: %s", err)
	}
	
	log.Println("Service registered with Consul")
}
```

## Interview Preparation Questions
1. **Why do we need service discovery in a microservices architecture?** (Dynamic IPs, scalability, health checking).
2. **Compare Client-Side vs. Server-Side service discovery.** (Focus on complexity vs. network hops).
3. **What is a Service Registry?** (The central database of service instances).
4. **How does health checking relate to service discovery?** (Preventing routing to failed instances).
5. **Which tool would you choose for a Kubernetes-native environment?** (Etcd, as it is built-in, or Consul if external service discovery is needed).

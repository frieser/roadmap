#AWS
#Cloud

---
tags: ['aws', 'roadmap', 'cloud', 'load-balancing', 'auto-scaling']
---

## Summary
Elastic Load Balancing (ELB) automatically distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, IP addresses, and Lambda functions. It ensures high availability and fault tolerance by routing traffic only to healthy targets. AWS provides four types of load balancers: Application Load Balancer (ALB), Network Load Balancer (NLB), Gateway Load Balancer (GLB), and Classic Load Balancer (CLB).

## Detailed Explanation

### Layer 7 vs Layer 4
The primary difference between the load balancer types is the OSI layer at which they operate:

- **Application Load Balancer (ALB) - Layer 7**: Operates at the Application layer. It is aware of the content of the requests (HTTP/HTTPS). This allows for advanced routing features like path-based (`/api`), host-based (`example.com`), header-based, or query-string-based routing. It also supports WebSockets and HTTP/2.
- **Network Load Balancer (NLB) - Layer 4**: Operates at the Transport layer. It is connection-based and handles TCP, UDP, and TLS. It is designed for ultra-high performance and can handle millions of requests per second with extremely low latency. It provides a static/Elastic IP per Availability Zone.
- **Classic Load Balancer (CLB) - Layer 4 & 7**: The legacy load balancer that operates at both layers. It lacks many of the modern features like target groups and advanced routing. It is primarily used for applications built within the EC2-Classic network.

### Listeners
A **Listener** is a process that checks for connection requests. It is configured with a protocol and a port (e.g., HTTPS on port 443). The rules defined for a listener determine how the load balancer routes requests to its registered targets. Each rule consists of a priority, one or more actions, and one or more conditions.

### Target Groups
**Target Groups** are used to route requests to one or more registered targets.
- **Targets**: Can be EC2 instances, microservices (containers), IP addresses, or Lambda functions.
- **Health Checks**: Performed at the target group level. The load balancer monitors the health of its targets and only routes traffic to healthy ones.
- **Routing**: When a listener rule condition is met, the traffic is forwarded to the specified target group.

### Comparison Table

| Feature | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Classic Load Balancer (CLB) |
| :--- | :--- | :--- | :--- |
| **OSI Layer** | Layer 7 (Application) | Layer 4 (Transport) | Layer 4/7 (Legacy) |
| **Protocols** | HTTP, HTTPS, gRPC, WebSockets | TCP, UDP, TLS | TCP, SSL, HTTP, HTTPS |
| **Routing Features** | Path, Host, Header, Query string | IP protocol, Source IP, Port | None (Basic) |
| **Performance** | High | Ultra-High (Millions of RPS) | Moderate |
| **Static / Elastic IP** | No (uses DNS name) | Yes (Static IP per AZ) | No |
| **Target Types** | Instance, IP, Lambda | Instance, IP, ALB | Instance only |
| **Health Checks** | HTTP, HTTPS, gRPC | TCP, HTTP, HTTPS | TCP, HTTP, HTTPS |
| **Security Groups** | Yes | Yes | Yes |

## Interview Questions

1. **Q: When should you choose an ALB over an NLB?**
   **A:** Choose ALB when you need advanced routing based on HTTP content (paths, hostnames), or when you need to integrate with AWS WAF, use Lambda as a target, or support modern protocols like gRPC and HTTP/2. NLB is the better choice for high-performance TCP/UDP traffic, ultra-low latency, or when the application requires a static IP address.

2. **Q: How does ALB handle sticky sessions and what are the limitations?**
   **A:** ALB supports sticky sessions (session affinity) using duration-based or application-based cookies. This ensures that subsequent requests from the same client are routed to the same target within a target group. However, if the target becomes unhealthy or the target group is scaled down, the session stickiness may be broken.

3. **Q: Can you use an NLB to load balance an ALB? Why would you do this?**
   **A:** Yes, you can register an ALB as a target for an NLB. This architectural pattern is often used to provide a static IP address (from the NLB) to an application that requires the advanced Layer 7 routing features of the ALB.

4. **Q: What is the purpose of "Deregistration Delay" (Connection Draining)?**
   **A:** Deregistration delay (or connection draining in CLB) allows the load balancer to complete in-flight requests before a target is removed from the target group or marked as unhealthy. This prevents abrupt connection termination for users during scaling events or maintenance.

5. **Q: Explain the difference between "Internet-facing" and "Internal" load balancers.**
   **A:** An **Internet-facing** load balancer has a public DNS name and routes requests from clients over the internet to targets. An **Internal** load balancer has only a private DNS name and routes requests from clients with access to the VPC to targets within the same or peered VPCs.

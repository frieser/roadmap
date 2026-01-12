---
tags: ['tools', 'roadmap']
---

# External Access to Services

## Summary
Exposing Kubernetes services to the external world is a fundamental requirement for most applications. Kubernetes provides several mechanisms for this, ranging from basic internal-only access (ClusterIP) to cloud-native external load balancers (LoadBalancer) and sophisticated L7 traffic management (Ingress). Choosing the right method depends on the environment (cloud vs. bare metal), security requirements, and the need for advanced features like SSL termination or path-based routing.

## Detailed Explanation

### 1. ClusterIP (Internal Only)
The default Service type. It exposes the Service on a cluster-internal IP. This type makes the Service only reachable from within the cluster.
*   **Use case:** Backend databases, internal microservices.

### 2. NodePort
Exposes the Service on each Node's IP at a static port (default range: 30000-32767). A `ClusterIP` Service is automatically created, and the `NodePort` service routes to it.
*   **Use case:** Quick demos, non-production environments, or when direct node access is required.
*   **Drawback:** Clients need to know Node IPs; limited port range.

### 3. LoadBalancer
Exposes the Service externally using a cloud provider's Load Balancer (e.g., AWS NLB, GCP Load Balancer). It automatically provisions a NodePort and ClusterIP.
*   **Use case:** Production web applications on public clouds.
*   **Drawback:** Can get expensive (one LB per service).

### 4. Ingress
Not a Service type, but an API object that manages external access to HTTP/HTTPS services. It acts as a smart router/reverse proxy (L7) capable of path-based routing (`/api` vs `/web`), SSL termination, and virtual hosting.
*   **Use case:** Exposing multiple services under a single IP/LoadBalancer to save costs and centralize management.

### Traffic Flow Diagram
```mermaid
graph TD
    User([External User])
    
    subgraph "L7 Layer (Ingress)"
        ING[Ingress Controller (NGINX/Traefik)]
    end
    
    subgraph "L4 Layer (Service)"
        LB[Cloud LoadBalancer]
        NP[NodePort]
        CIP[ClusterIP]
    end
    
    subgraph "Pod Layer"
        P1[Pod: Go App]
        P2[Pod: Go App]
    end

    User -->|HTTPS| LB
    LB -->|Traffic| ING
    ING -->|Routing Rules| CIP
    CIP -->|Selector| P1
    CIP -->|Selector| P2
    
    style ING fill:#f9f,stroke:#333
    style LB fill:#bbf,stroke:#333
```

---

## Go Application

### Production-Ready Go Server
When exposing a Go app externally, you need to handle graceful shutdowns and health checks to play nicely with Load Balancers.

```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, External World!")
	})
	// Health check for LoadBalancer probes
	mux.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(200)
		w.Write([]byte("ok"))
	})

	srv := &http.Server{
		Addr:    ":8080",
		Handler: mux,
	}

	go func() {
		fmt.Println("Starting server on :8080")
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			panic(err)
		}
	}()

	// Graceful Shutdown
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := srv.Shutdown(ctx); err != nil {
		fmt.Printf("Server forced to shutdown: %v", err)
	}
	fmt.Println("Server exiting")
}
```

### Kubernetes Manifests (Service + Ingress)

```yaml
# 1. The Service (Internal Abstraction)
apiVersion: v1
kind: Service
metadata:
  name: go-app-svc
spec:
  selector:
    app: go-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
# 2. The Ingress (External Exposure)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-app-ingress
spec:
  rules:
  - host: my-go-app.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-app-svc
            port:
              number: 80
```

---

## Interview Questions

### 1. What is the difference between `NodePort` and `LoadBalancer`?
**Answer**: `NodePort` opens a specific port on **every Node** in the cluster, allowing direct access via any Node's IP. `LoadBalancer` builds on top of NodePort by requesting the Cloud Provider to provision an external Load Balancer (like an AWS ELB) that routes traffic to those NodePorts, providing a single stable external IP.

### 2. Can you have an Ingress without a Service?
**Answer**: No. Ingress is a set of routing rules that directs traffic to **backend Services**. It does not point directly to Pods. The Ingress Controller uses the Service to discover the underlying Pod endpoints.

### 3. What is "Ingress Class"?
**Answer**: An `IngressClass` allows you to support multiple Ingress Controllers in the same cluster (e.g., one NGINX controller for public traffic and another for internal tools). You specify the `ingressClassName` in your Ingress manifest to tell Kubernetes which controller should implement the rules.

### 4. How does SSL/TLS termination work in Ingress?
**Answer**: You create a Kubernetes `Secret` (type `kubernetes.io/tls`) containing your cert and key. Then, in the Ingress manifest, you reference this secret under the `tls` section. The Ingress Controller (like NGINX) handles the decryption at the edge and forwards plain HTTP traffic to your backend Pods.

### 5. Why is `ClusterIP` the default Service type?
**Answer**: Most services in a microservices architecture are internal (databases, caches, helper services) and should not be exposed to the public internet. `ClusterIP` provides a secure, internal-only VIP that keeps the attack surface minimal by default.

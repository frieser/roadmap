---
---

# Linkerd

Linkerd is a "service mesh for Kubernetes." It positions itself as the ultra-lightweight, simple, and fast alternative to Istio.

## Summary

Linkerd uses a specialized "micro-proxy" written in **Rust** (called `linkerd2-proxy`) instead of Envoy. This results in significantly lower memory and CPU footprint. Its philosophy is "It just works"—providing mTLS and golden metrics (Success Rate, Latency, Throughput) with almost zero configuration.

## Detailed Explanation

### 1. Architecture
*   **Control Plane**: Written in Go. Runs in the `linkerd` namespace. Manages identity and telemetry.
*   **Data Plane**: The Rust proxy sidecars. They are injected into application pods.

### 2. Features
*   **Automatic mTLS**: Enabled by default. No complex setup.
*   **Traffic Split**: Supports the SMI (Service Mesh Interface) standard for Canary deployments.
*   **Observability**: Provides a rich dashboard (`linkerd viz`) and CLI tools (`linkerd top`, `linkerd tap`) to see live traffic.

---

## Go Implementation Example

Linkerd is transparent. You write standard Go gRPC or HTTP services. Linkerd handles the load balancing.

### Standard gRPC Service (Managed by Linkerd)
Linkerd automatically detects HTTP/2 and gRPC traffic and load balances it (which Kubernetes Service load balancing *cannot* do effectively at L7).

```go
package main

import (
	"context"
	"log"
	"net"

	"google.golang.org/grpc"
	pb "my-app/proto"
)

type server struct {
	pb.UnimplementedGreeterServer
}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	// Linkerd adds metadata to the context (like latency info) but it's invisible here.
	// We just focus on logic.
	return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{})
	
	// When running in the mesh, Linkerd's proxy intercepts this port
	// and handles mTLS and load balancing.
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

## Interview Questions

**Q: Why use Linkerd over Istio?**
**A:** Use Linkerd if your priority is **Simplicity** and **Performance**. If you don't need complex enterprise features (like external auth integration, complex VM support, or non-k8s workloads), Linkerd is much easier to install and maintain. Its Rust proxy is arguably faster and lighter than Envoy.

**Q: Why does Kubernetes natively struggle with gRPC Load Balancing?**
**A:** gRPC uses HTTP/2, which multiplexes many requests over a single TCP connection. Kubernetes Service (L4 Load Balancer) operates at the connection level. Once a connection is established to one Pod, all subsequent requests go to that same Pod, leading to uneven load. Linkerd (L7 Proxy) understands the individual requests inside the TCP connection and can balance them across all available Pods.

**Q: What is `linkerd tap`?**
**A:** `linkerd tap` is a CLI tool that allows you to listen to a live stream of requests to/from a service (like `tcpdump` for microservices). It shows you the method, path, latency, and status code of real-time traffic, which is invaluable for debugging.

---
---

## Summary
Kubernetes (K8s) is an open-source container orchestration platform that automates deploying, scaling, and managing containerized applications. It has become the de facto standard for cloud-native infrastructure, replacing older tools like Docker Swarm due to its robust ecosystem and extensibility.

## Detailed Explanation

### Core Concepts
1.  **Pod**: The smallest deployable unit. Wraps one or more containers (e.g., App + Sidecar).
2.  **Deployment**: Manages Pods (Scaling, Rolling Updates, Rollbacks).
3.  **Service**: A stable network endpoint (Load Balancer) to access a set of Pods.
4.  **Ingress**: Manages external access (HTTP/HTTPS routes) to services.

### Why K8s won
*   **Declarative**: You define "I want 3 replicas", and K8s makes it happen (Self-healing).
*   **Extensible**: CRDs (Custom Resource Definitions) allow extending the API.
*   **Ecosystem**: Huge community support (Helm, Prometheus, Istio).

## Go-Specific Context/Examples

Kubernetes is written in **Go**. Its entire ecosystem revolves around Go.

### Go Client (`client-go`)
Writing a K8s controller in Go involves watching resources and reconciling state.

```go
package main

import (
	"context"
	"fmt"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// Use local kubeconfig
	config, _ := clientcmd.BuildConfigFromFlags("", "/home/user/.kube/config")
	clientset, _ := kubernetes.NewForConfig(config)

	// List Pods
	pods, _ := clientset.CoreV1().Pods("default").List(context.TODO(), metav1.ListOptions{})
	for _, p := range pods.Items {
		fmt.Printf("Pod: %s\n", p.Name)
	}
}
```

## Interview Questions

**Q: What is the difference between a Pod and a Container?**
**A:** A Container is the runtime (Docker). A Pod is a K8s object that wraps containers. Pods provide shared storage (Volumes) and networking (IP address) to the containers inside. Containers in the same Pod share localhost.

**Q: Explain the "Reconciliation Loop".**
**A:** It is the core logic of K8s controllers. The controller watches the **Current State** (what's running) and compares it to the **Desired State** (what's in YAML). If they differ (e.g., a pod died), it takes action to make them match (starts a new pod).

**Q: Why use a Service instead of talking to Pod IPs directly?**
**A:** Pods are ephemeral. If a pod dies and restarts, it gets a new IP. A Service provides a stable Virtual IP (ClusterIP) and DNS name that load balances traffic to whatever dynamic pods typically match the selector.

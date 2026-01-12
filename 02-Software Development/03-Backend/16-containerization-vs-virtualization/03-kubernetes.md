---
---

# Kubernetes (K8s)

Kubernetes is the industry standard for container orchestration. It automates the deployment, scaling, and management of containerized applications.

## 1. Architecture: Control Plane vs. Worker

### Control Plane (Brain)
*   **API Server**: The only component with external access. Validates and processes REST requests.
*   **Etcd**: Consistent, distributed Key-Value store. Holds the **entire cluster state**. Uses Raft consensus.
*   **Scheduler**: Assigns unscheduled pods to nodes based on resource availability and constraints (Taints/Tolerations).
*   **Controller Manager**: Runs control loops (e.g., ReplicaSet Controller ensures 3 pods are running if you asked for 3).

### Worker Node (Muscle)
*   **Kubelet**: Agent running on the node. Talks to the API server and manages container lifecycles via the Container Runtime Interface (CRI).
*   **Kube-proxy**: Manages network rules (iptables/IPVS) to allow service communication.
*   **Container Runtime**: The software running the containers (containerd, CRI-O).

## 2. Networking Models

*   **CNI (Container Network Interface)**: Plugin standard for connectivity.
    *   **Requirement**: Every Pod gets its own IP. All Pods can talk to all other Pods without NAT.
*   **Service Mesh**: Using a Sidecar Proxy (Envoy) to handle mTLS, observability, and traffic splitting (Canary) at the application layer.
*   **Ingress vs. Gateway API**:
    *   **Ingress**: Basic HTTP/HTTPS routing.
    *   **Gateway API**: Modern, role-based API supporting TCP/UDP, traffic splitting, and advanced routing.

## 3. Advanced Patterns

*   **Sidecar**: A helper container in the same Pod (e.g., logging agent, proxy).
*   **Operator**: A custom controller that extends the K8s API to manage complex stateful applications (e.g., "PostgresOperator" handles backups and failover).
*   **Ambassador**: Proxying connections to the outside world.

## 4. Go Integration (Client-go & Controllers)

Kubernetes is written in Go. Extending it requires deep knowledge of the Go ecosystem.

### Client-go
Using the official client to interact with the cluster.

```go
clientset, _ := kubernetes.NewForConfig(config)
pods, _ := clientset.CoreV1().Pods("default").List(ctx, metav1.ListOptions{})
```

### Writing an Operator (Kubebuilder)
The **Reconcile Loop** is the heart of any controller.

```go
// Reconcile moves current state -> desired state
func (r *MyReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Fetch CRD
    // 2. Check if Child Resources (e.g. Deployments) exist
    // 3. Create/Update them if they don't match Spec
    return ctrl.Result{}, nil
}
```

## 5. Interview Questions

**Q: How does Etcd ensure consistency?**
**A:** It uses the **Raft Consensus Algorithm**. A write is only committed once a quorum ($N/2 + 1$) of nodes have acknowledged it. This ensures that the cluster state is strongly consistent even during network partitions.

**Q: Iptables vs IPVS mode in Kube-proxy?**
**A:** `iptables` rules are evaluated sequentially ($O(N)$), which becomes slow with thousands of services. `IPVS` uses hash tables ($O(1)$) and supports advanced load balancing algorithms, making it better for large-scale clusters.

**Q: What happens when a Pod is deleted? (Graceful Shutdown)**
**A:**
1.  API server marks Pod as "Terminating".
2.  PreStop hook runs.
3.  SIGTERM is sent to PID 1.
4.  Kubelet waits for `terminationGracePeriodSeconds` (default 30s).
5.  SIGKILL is sent if process is still running.
**Crucial**: Your app must handle SIGTERM to close connections properly!

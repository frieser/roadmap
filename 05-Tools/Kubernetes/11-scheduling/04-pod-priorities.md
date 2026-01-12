---
---

## Summary
Pod Priority and Preemption is a Kubernetes feature that ensures the most critical workloads (like system components or payment processors) get scheduled even if the cluster is full. It allows the scheduler to evict (preempt) lower-priority pods to make room for higher-priority ones.

## Detailed Explanation

### PriorityClass
A non-namespaced object defining a priority integer.
*   **SystemCritical**: > 2 Billion (Reserved).
*   **HighPriority**: 1,000,000.
*   **Default**: 0.

### Preemption Logic
1.  **Pending**: A high-priority Pod is pending because no nodes have capacity.
2.  **Search**: Scheduler checks nodes to see if removing low-priority pods creates enough space.
3.  **Eviction**: Scheduler deletes lower-priority pods on a chosen node.
4.  **Scheduling**: The high-priority Pod is scheduled.

## Go-Specific Context/Examples

Defining a PriorityClass in YAML (or Go structs for K8s operators).

### YAML Example
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-service
value: 1000000
globalDefault: false
description: "For critical backend services only."
```

### Assigning to Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-go-app
spec:
  priorityClassName: critical-service
  containers: ...
```

## Interview Questions

**Q: What happens to the preempted (evicted) pods?**
**A:** They are terminated gracefully (SIGTERM). If they are managed by a Controller (Deployment/StatefulSet), they return to the queue and try to be rescheduled elsewhere. If the cluster is totally full, they stay Pending.

**Q: Can a Pod preempt another Pod with the same priority?**
**A:** No. Preemption only happens if the pending Pod has a **strictly higher** priority than the running Pods.

**Q: Why limit the use of high priority?**
**A:** If everyone sets their pods to "High Priority", the feature becomes useless. It creates a "Priority Inflation" where nothing can be preempted, and critical system pods might fail to start during an outage.
